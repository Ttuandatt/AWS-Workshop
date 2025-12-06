HƯỚNG DẪN DEPLOY BACKEND MICROSERVICES TRÊN AWS ECS (FINAL VERSION)
===================================================================

🏗 TỔNG QUAN HỆ THỐNG
---------------------

-   **Region:** Singapore (`ap-southeast-1`).

-   **Frontend Domain:** `https://sgutodolist.com`.

-   **Backend Domain:** `https://api.sgutodolist.com`.

-   **Kiến trúc Mạng (Tiết kiệm chi phí - Không dùng NAT Gateway):**

    -   **Public Subnets:** Chứa tất cả ECS Tasks (App, Kafka, AI) + ALB + Bastion. (Bắt buộc để Task có thể tải Image từ Internet).

    -   **Private Subnets:** Chứa RDS + Redis (Để bảo mật dữ liệu).

* * * * *

GIAI ĐOẠN 1: CHUẨN BỊ NETWORK & SECURITY
----------------------------------------

### 1.1 Tạo VPC & Subnets

1.  Vào **VPC Console** > **Create VPC**.

2.  Chọn **VPC and more**.

    -   **Name tag auto-generation:** Nhập `SGU-Microservices`.

    -   **IPv4 CIDR block:** `10.0.0.0/16`.

    -   **Number of Availability Zones (AZs):** Chọn **2** (`ap-southeast-1a` và `ap-southeast-1b`).

    -   **Number of public subnets:** Chọn **2**.

    -   **Number of private subnets:** Chọn **2**.

    -   **NAT gateways:** Chọn **None** (⚠️ Quan trọng để tiết kiệm ~$30/tháng).

    -   **VPC endpoints:** Chọn **None**.

    -   **DNS options:** Tích chọn cả **Enable DNS hostnames** và **Enable DNS resolution**.

3.  Click **Create VPC**.

### 1.2 Tạo Security Groups (Tường lửa)

Vào **VPC** > **Security Groups** > **Create security group**. Tạo lần lượt 4 nhóm:

**1\. `public-alb-sg`** (Cho Load Balancer):

-   Description: `Security group for SGUTODOLIST ALB`.

-   VPC: `SGU-Microservices-VPC`.

-   **Inbound rules:**

    -   Type: `HTTPS` (443) | Source: `0.0.0.0/0`.

    -   Type: `HTTP` (80) | Source: `0.0.0.0/0`.

**2\. `ecs-app-sg`** (Cho các Service Container):

-   Description: `Security group for SGUTODOLIST Service Container`.

-   VPC: `SGU-Microservices-VPC`.

-   **Inbound rules (Bước 1):**

    -   Type: `Custom TCP` | Port: `8080` | Source: Chọn `public-alb-sg`.

    -   Type: `Custom TCP` | Port: `8081` | Source: Chọn `public-alb-sg`.

    -   Type: `Custom TCP` | Port: `8082` | Source: Chọn `public-alb-sg`.

    -   Type: `Custom TCP` | Port: `9998` | Source: Chọn `public-alb-sg`.

    -   Type: `Custom TCP` | Port: `9999` | Source: Chọn `public-alb-sg`.

    -   Type: `Custom TCP` | Port: `9997` | Source: Chọn `public-alb-sg` (Cho AI Service).

-   *Nhấn **Create security group** trước.*

-   **Inbound rules (Bước 2 - Self referencing):**

    -   Chọn lại `ecs-app-sg` > Edit inbound rules > Add rule.

    -   Type: `All TCP` | Port: `0-65535` | Source: Tìm và chọn chính `ecs-app-sg`.

**3\. `private-db-sg`** (Cho RDS & Redis & Kafka):

-   Description: `Security group for SGUTODOLIST RDS & Redis & Kafka`.

-   VPC: `SGU-Microservices-VPC`.

-   **Inbound rules:**

    -   Type: `MySQL` (3306) | Source: Chọn `ecs-app-sg`.

    -   Type: `Custom TCP` (Port `6379`) | Source: Chọn `ecs-app-sg` (Redis).

    -   Type: `Custom TCP` (Port `9092`) | Source: Chọn `ecs-app-sg` (Kafka).

**4\. `bastion-sg`** (Cho máy nhảy):

-   Description: `Security group for SGUTODOLIST bastion`.

-   VPC: `SGU-Microservices-VPC`.

-   **Inbound rules:**

    -   Type: `SSH` (22) | Source: `My IP` (IP máy của bạn).

* * * * *

GIAI ĐOẠN 2: THIẾT LẬP TÊN MIỀN & SSL
-------------------------------------

1.  Đổi Region sang **Singapore (ap-southeast-1)**.

2.  Vào **Certificate Manager (ACM)**.

3.  **Request a certificate** > **Request a public certificate**.

4.  Domain names: `api.sgutodolist.com`.

5.  Validation method: **DNS validation**.

6.  Click **Request**.

7.  Click vào Certificate ID > Phần **Domains** > **Create records in Route 53**.

8.  Đợi trạng thái chuyển sang **Issued** (Màu xanh).

* * * * *

GIAI ĐOẠN 3: CẤU HÌNH INFRASTRUCTURE
------------------------------------

### 3.1 RDS MySQL

1.  Vào **RDS** > **Create database** > **Standard create** > **MySQL**.

2.  Template: **Free tier**.

3.  Identifier: `sgu-todolist-db`.

4.  Username: `root` / Password: `12345678` (Ví dụ).

5.  **Connectivity:**

    -   Compute resource: **Don't connect to an EC2...**.

    -   VPC: `SGU-Microservices-VPC`.

    -   Public access: **No**.

    -   VPC security group: Chọn `private-db-sg` (bỏ default).

    -   Availability Zone: **`ap-southeast-1a`**.

6.  Click **Create database**.

7.  *Sau khi tạo xong, copy **Endpoint** của database.*

### 3.2 ElastiCache Redis (Cấu hình chi tiết)

1.  Vào **Redis OSS caches** > **Create Redis OSS cache**.

#### Step 1: SETTINGS

-   **Cluster settings:**

    -   Engine: **Redis OSS**.

    -   Deployment option: **Node-based cluster**.

    -   Creation method: **Cluster cache**.

-   **Cluster info:**

    -   Cluster mode: **Disabled**.

    -   Name: `sgu-redis`.

    -   Multi-AZ: **Uncheck (Disable)**.

    -   Auto-failover: **Uncheck (Disable)**.

-   **Cache settings:**

    -   Node type: **`cache.t3.micro`**.

    -   Number of replicas: **0**.

-   **Connectivity:**

    -   Subnet groups: **Create a new subnet group**.

        -   Name: `sgu-redis-subnet-group`.

        -   VPC ID: `SGU-Microservices-VPC`.

        -   Selected subnets: Chọn 2 **Private Subnets**.

    -   Availability Zone placements: **Specify Availability Zones**.

        -   Primary: **`ap-southeast-1a`**.

#### Step 2: ADVANCED SETTINGS

-   **Security:**

    -   Selected security groups: Chọn **`private-db-sg`** (Bỏ default).

-   **Backup:** Enable automatic backups: **Uncheck (Disable)**.

-   **Logs:** Disable hết.

*(Nhấn Create. Sau khi xong, copy **Primary Endpoint**).*

### 3.3 Kafka (Docker trên ECS)

**Bước 1: Tạo Namespace**

1.  Vào **Cloud Map** > **Create namespace**.

2.  Name: `sgu.local`.

3.  Instance discovery: **API calls and DNS queries in VPCs**.

4.  VPC: `SGU-Microservices-VPC`.

5.  Create.

**Bước 2: Tạo Task Definition cho Kafka**

1.  Vào **Amazon ECS** > **Task definitions**.

2.  Chọn `kafka-server-td` (nếu đã có) > **Create new revision** (hoặc Create new nếu chưa có).

3.  **Infrastructure:**

    -   Launch type: **AWS Fargate**.

    -   CPU: `.5 vCPU`.

    -   Memory: `1 GB` (Kafka cần ít nhất 1GB để khởi động Java Heap).

    -   Task Execution Role: `ecsTaskExecutionRole`.

4.  **Container - 1:**

    -   **Name:** `kafka-server`

    -   **Image URI:** `bitnami/kafka:latest`

    -   **Container Port:** `9092`

    -   **Protocol:** `TCP`

5.  **Environment Variables (Quan trọng - Hãy xóa cũ và nhập lại y hệt bảng này):**

| **Key** | **Value** | **Giải thích (Không cần nhập)** |
| --- | --- | --- |
| `KAFKA_CFG_NODE_ID` | `0` | Định danh node |
| `KAFKA_CFG_PROCESS_ROLES` | `controller,broker` | Vừa quản lý vừa chứa dữ liệu |
| `KAFKA_CFG_LISTENERS` | `PLAINTEXT://:9092,CONTROLLER://:9093` | Lắng nghe nội bộ |
| `KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP` | `CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT` | Protocol map |
| `KAFKA_CFG_CONTROLLER_QUORUM_VOTERS` | `0@127.0.0.1:9093` | Tự trỏ về chính nó (localhost) |
| `KAFKA_CFG_ADVERTISED_LISTENERS` | `PLAINTEXT://kafka.sgu.local:9092` | **Địa chỉ để Service khác gọi vào** |
| `KAFKA_CFG_LOG_DIRS` | `/tmp/kafka-logs` | **FIX LỖI CRASH:** Ghi log vào thư mục tạm |
| `ALLOW_PLAINTEXT_LISTENER` | `yes` | Cho phép kết nối không cần SSL |

6.  **Logging:** > **Use log collection** (Để xem lỗi nếu có).

7.  Nhấn **Create**.

**Bước 3: Deploy Kafka Service**

Sau khi có Task Definition mới, ta tiến hành tạo Service.

1.  Vào Cluster > Tab **Services** > **Create**.

2.  **Service details:**

    -   **Task definition family:** Chọn `kafka-server-td` (Chọn Revision mới nhất vừa tạo ở Bước 1).

    -   **Service name:** Nhập `kafka-server`.

    -   **Desired tasks:** `1`.

3.  **Networking (Bắt buộc):**

    -   **VPC:** `SGU-Microservices-VPC`.

    -   **Subnets:** Chọn **2 PUBLIC SUBNETS**. (Cần Internet để tải Image Kafka về).

    -   **Security group:** Chọn `private-db-sg`.

    -   **Public IP:** **Turned on**.

4.  **Load Balancing:**

    -   **KHÔNG CHỌN (Uncheck)**. Kafka chỉ dùng nội bộ, không cần Load Balancer.

5.  **Service Discovery (Rất quan trọng):**

    -   Tích chọn ✅ **Use service discovery**.

    -   **Namespace:** Chọn `sgu.local`.

    -   **Service discovery name:** Nhập chính xác chữ: **`kafka`**.

    -   *(Hệ thống sẽ tạo DNS: `kafka.sgu.local`. Các service Auth/User/Taskflow sẽ kết nối qua địa chỉ này).*

6.  Nhấn **Create**.

### 3.4 Khởi tạo Database (Chi tiết)

**1\. Update Security Group:**

-   Vào `private-db-sg` > Inbound Rules > Add: `MySQL (3306)` | Source: `bastion-sg`.

**2\. Tạo Bastion Host (EC2):**

-   Launch Instance > Name `sgu-bastion` > AMI **Amazon Linux 2023** > Type `t3.micro`.

-   Key pair: Tạo mới `sgutodolist-key` (tải về máy).

-   VPC: `SGU-Microservices-VPC`.

-   Subnet: **Public Subnet 1**.

-   Auto-assign public IP: **Enable** (Bắt buộc).

-   Security groups: Chọn `bastion-sg`.

**3\. Init Database (Chạy lệnh từ máy Local):**

-   Mở Terminal VS Code tại thư mục chứa `sgu-key.pem`.

-   **Bước 1: SSH vào Bastion**

    PowerShell

    ```
    # Với Windows
    icacls "sgutodolist-key.pem" /inheritance:r
    icacls "sgutodolist-key.pem" /grant:r "$($env:USERNAME):(R)"
    ssh -i "sgu-key.pem" ec2-user@<PUBLIC-IP-BASTION> (Public IPv4 address của máy chủ EC2 sgu-bastion)


    # Với Mac/Linux
    chmod 400 sgu-key.pem
    ssh -i sgu-key.pem ec2-user@<PUBLIC-IP-BASTION>

    ```

-   **Bước 2: Cài MySQL Client (trên Bastion)**

    ```bash
    sudo dnf install mariadb105 -y

    ```

-   **Bước 3: Kết nối RDS**

    Bash

    ```
    mysql -h sgu-todolist-db.cnsww4so4xfb.ap-southeast-1.rds.amazonaws.com -u root -p
    sgu-todolist-db.cnsww4so4xfb.ap-southeast-1.rds.amazonaws.com
    # Nhập password: 12345678

    ```

-   **Bước 4: Tạo Database (SQL)**

    SQL

    ```
    CREATE DATABASE aws_todolist_database CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
    SHOW DATABASES;
    EXIT;

    ```

-   **Bước 5:** Gõ `exit` để thoát khỏi Bastion.

* * * * *

GIAI ĐOẠN 4: CẬP NHẬT CODE & BUILD IMAGE
----------------------------------------

### 4.1 Cấu hình API Gateway (Quan trọng)

Ta cần làm 2 việc tại source code của **API Gateway**:

**Bước 1: Tạo file cấu hình CORS (Java)** File này giúp xử lý CORS toàn cục một cách chính xác trên môi trường Reactive.

-   **File:** `src/main/java/aws/todolist/api_gateway/config/CorsConfig.java`

-   **Nội dung:**

Java

```
package aws.todolist.api_gateway.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.reactive.CorsWebFilter;
import org.springframework.web.cors.reactive.UrlBasedCorsConfigurationSource;
import java.util.Arrays;

@Configuration
public class CorsConfig {

    @Bean
    public CorsWebFilter corsWebFilter() {
        CorsConfiguration corsConfig = new CorsConfiguration();

        // 1. Cho phép các Domain này gọi API
        corsConfig.setAllowedOrigins(Arrays.asList(
            "https://sgutodolist.com",
            "https://www.sgutodolist.com",
            "http://localhost:3000", // Frontend Dev
            "http://localhost:4200", // Frontend Dev (Backup)
            "http://localhost:5500"
        ));

        // 2. Cấu hình method và header
        corsConfig.setMaxAge(3600L);
        corsConfig.addAllowedMethod("*"); // GET, POST, PUT, DELETE...
        corsConfig.addAllowedHeader("*");
        corsConfig.setAllowCredentials(true);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", corsConfig);

        return new CorsWebFilter(source);
    }
}

```

**Bước 2: Cập nhật `application.yml` (Chuẩn hóa biến môi trường)*

-   **File:** `src/main/resources/application.yml`

YAML

```
server:
  port: 8080

jwt:
  secret: 843567893696976453275974432697R634976R738467TR678T34865R6834R8763T478378637664538745673865783678548735687R3

spring:
  application:
    name: api-gateway

  cloud:
    gateway:
      # ⚠️ Đã tắt Global CORS ở đây để dùng file Java CorsConfig.java (tránh xung đột)
      # globalcors: ... (Đã comment)

      # 🔹 Routes Configuration (Sử dụng biến môi trường từ ECS Task Def)
      routes:
        - id: ai-model-service
          uri: ${AI_MODEL_SERVICE:http://localhost:9997}
          predicates:
            - Path=/api/model/**

        - id: user-service
          uri: ${USER_SERVICE_URL:http://localhost:8081}
          predicates:
            - Path=/api/user/**

        - id: taskflow-service
          uri: ${TASKFLOW_SERVICE_URL:http://localhost:8082}
          predicates:
            - Path=/api/taskflow/**

        - id: auth-service
          uri: ${AUTH_SERVICE_URL:http://localhost:9999}
          predicates:
            - Path=/api/auth/**

        # 🔹 Notification Routes
        - id: notification-ws
          uri: ${NOTIFICATION_SERVICE_URL:http://localhost:9998}
          predicates:
            - Path=/api/notification/ws/**

        - id: notification-rest
          uri: ${NOTIFICATION_SERVICE_URL:http://localhost:9998}
          predicates:
            - Path=/api/notification/**

  # 🧱 Redis Configuration (Sử dụng biến môi trường)
  data:
    redis:
      host: ${SPRING_DATA_REDIS_HOST:redis}
      port: ${SPRING_DATA_REDIS_PORT:6379}
      timeout: 6000
      lettuce:
        pool:
          max-active: 8
          max-idle: 8
          min-idle: 0
          max-wait: -1ms

# =========================
# ⚙️ Management & Monitoring
# =========================
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    health:
      show-details: always

# =========================
# 🧭 SpringDoc Swagger UI
# =========================
springdoc:
  api-docs:
    enabled: true
  swagger-ui:
    urls:
      # Sử dụng biến môi trường để Swagger hoạt động đúng trên Domain thật
      - name: user-service
        url: ${USER_SERVICE_URL:http://localhost:8081}/api/user/v3/api-docs
      - name: taskflow-service
        url: ${TASKFLOW_SERVICE_URL:http://localhost:8082}/api/taskflow/v3/api-docs
      - name: auth-service
        url: ${AUTH_SERVICE_URL:http://localhost:9999}/api/auth/v3/api-docs
      - name: notification-service
        url: ${NOTIFICATION_SERVICE_URL:http://localhost:9998}/api/notification/v3/api-docs
```
* * * * *

### 4.2 Build & Push 6 Images lên ECR

Mở Terminal tại thư mục gốc của dự án (`todolist-backend`), chạy lần lượt các lệnh sau:

**1\. Đăng nhập ECR:**

Bash

```
aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com

```

**2\. Build & Push lần lượt 6 Services:**

-   **API Gateway:**

    ```bash
    cd api-gateway
    docker build -t api-gateway:latest .
    docker tag api-gateway:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/api-gateway:latest
    docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/api-gateway:latest
    cd ..

    ```

-   **Auth Service:**

    Bash

    ```
    cd auth-service
    docker build -t auth-service:latest .
    docker tag auth-service:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/auth-service:latest
    docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/auth-service:latest
    cd ..

    ```

-   **User Service:**

    Bash

    ```
    cd user-service
    docker build -t user-service:latest .
    docker tag user-service:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/user-service:latest
    docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/user-service:latest
    cd ..

    ```

-   **Taskflow Service:**

    Bash

    ```
    cd taskflow-service
    docker build -t taskflow-service:latest .
    docker tag taskflow-service:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/taskflow-service:latest
    docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/taskflow-service:latest
    cd ..

    ```

-   **Notification Service:**

    Bash

    ```
    cd notification-service
    docker build -t notification-service:latest .
    docker tag notification-service:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/notification-service:latest
    docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/notification-service:latest
    cd ..

    ```

-   **AI Model Service:** *Lưu ý: Thư mục chứa code AI của bạn tên là `model` trong docker-compose*

    Bash

    ```
    cd model
    docker build -t ai-model-service:latest .
    docker tag ai-model-service:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/ai-model-service:latest
    docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/ai-model-service:latest
    cd ..

    ```

* * * * *

GIAI ĐOẠN 5: CẤU HÌNH ALB & HTTPS
---------------------------------------

1.  **Target Groups:** Tạo 5 cái (`IP addresses`, `HTTP`).

    -   `api-gateway-tg`: 8080

    -   `auth-tg`: 9999

    -   `user-tg`: 8081

    -   `task-tg`: 8082

    -   `noti-tg`: 9998

    -   *(Không cần tạo TG cho AI Model vì Gateway gọi nội bộ).*

2.  **Load Balancer:**

    -   Tạo ALB `sgu-alb`. Scheme: **Internet-facing**.

    -   Subnets: **2 Public Subnets**. Security Group: `public-alb-sg`.

    -   Listener **HTTPS:443**: Forward to `tg-api-gateway`. Certificate `api.sgutodolist.com`.

3.  **Rules (HTTPS Listener):**

    -   Path `/api/auth/*` -> `tg-auth`.

    -   Path `/api/users/*` -> `tg-user`.

    -   Path `/api/tasks/*` -> `tg-task`.

    -   Path `/api/notifications/*` -> `tg-noti`.

    -   Default -> `tg-api-gateway`.

* * * * *

GIAI ĐOẠN 5: CẤU HÌNH ALB & HTTPS
---------------------------------------

1.  **Target Groups:** Tạo 5 cái (`IP addresses`, `HTTP`).


**Tạo 5 Target Groups**

Ta cần lặp lại quy trình dưới đây **5 lần** cho 5 service, chỉ thay đổi **Name** và **Port**.

**Quy trình chuẩn cho mỗi Target Group:**

**Step 1: Create target group**

-   **Target type:** Chọn **IP addresses** (Bắt buộc cho Fargate).

-   **Target group name:** Nhập tên tương ứng (ví dụ: `api-gateway-tg`).

-   **Protocol:** Chọn **HTTP**.

-   **Port:** Nhập port tương ứng (ví dụ: `8080`).

-   **IP address type:** **IPv4**.

-   **VPC:** Chọn `SGU-Microservices-vpc`.

-   **Protocol version:** **HTTP1**.

-   **Health checks:**

    -   **Health check protocol:** **HTTP**.

    -   **Health check path:** Nhập `/actuator/health` (Thay cho dấu `/` mặc định).

-   *Các phần khác (Advanced health check settings, Tags...):* Giữ nguyên.

-   Nhấn **Next**.

**Step 2: Register targets**

-   ⚠️ **QUAN TRỌNG:** Tại màn hình này, ta **KHÔNG LÀM GÌ CẢ**.

-   Không nhập IP, không nhấn "Include as pending".

-   *Lý do:* Khi ta Deploy ECS Service ở Giai đoạn 7, ECS sẽ tự động thêm IP của container vào đây.

-   Nhấn nút **Next**.

**Step 3: Review and create**

-   Nhấn nút **Create target group**.


**Danh sách thông số cho 5 Target Groups:**

Bạn hãy tạo lần lượt với các thông số sau (các thông số khác giống hệt hướng dẫn trên):

| **Số TT** | **Target Group Name** | **Port** | **Health Check Path** | **Ghi chú** |
| --- | --- | --- | --- | --- |
| 1 | **`api-gateway-tg`** | **8080** | `/actuator/health` | Cho API Gateway |
| 2 | **`auth-tg`** | **9999** | `/actuator/health` | Cho Auth Service |
| 3 | **`user-tg`** | **8081** | `/actuator/health` | Cho User Service |
| 4 | **`task-tg`** | **8082** | `/actuator/health` | Cho TaskFlow |
| 5 | **`noti-tg`** | **9998** | `/actuator/health` | Cho Notification |


### 2\. Tạo Application Load Balancer (ALB)

Vào **EC2 Console** > **Load Balancers** > **Create load balancer**.

**Step 1: Basic configuration**

-   **Load balancer type:** Chọn **Application Load Balancer**.

-   **Load balancer name:** `sgu-alb`.

-   **Scheme:** Chọn **Internet-facing** (Quan trọng: Để user từ internet truy cập được).

-   **IP address type:** **IPv4**.

**Step 2: Network mapping**

-   **VPC:** Chọn `SGU-Microservices-vpc`.

-   **Mappings (Availability Zones):** Bạn phải chọn 2 Zones và **2 Public Subnets**:

    -   Tích chọn **ap-southeast-1a** -> Chọn Subnet **Public 1** (Ví dụ: `...public1...`).

    -   Tích chọn **ap-southeast-1b** -> Chọn Subnet **Public 2** (Ví dụ: `...public2...`).

    -   *Lý do:* ALB phải nằm ở Public Subnet để nhận traffic từ Internet.

**Step 3: Security groups**

-   Bỏ chọn `default`.

-   Chọn **`public-alb-sg`**.

**Step 4: Listeners and routing (Tạo 2 Listener)**

-   **Listener 1 (HTTP:80):** (Có sẵn mặc định)

    -   Protocol: `HTTP` | Port: `80`.

    -   **Default action:** Chọn Forward to target group -> **`api-gateway-tg`**.

-   **Listener 2 (HTTPS:443):** (Bạn bấm nút **Add listener** để thêm dòng này)

    -   Protocol: `HTTPS` | Port: `443`.

    -   **Default action:** Chọn Forward to target group -> **`api-gateway-tg`**.

    -   **Secure listener settings:**

        -   **Certificate source:** ACM.

        -   **Certificate:** Chọn chứng chỉ `api.sgutodolist.com` (đã tạo ở Giai đoạn 2).

**Step 5: Review & Create**

-   Nhấn **Create load balancer**.

-   *Đợi khoảng 2-3 phút để State chuyển từ `Provisioning` sang `Active`.*

* * * * *

### 3\. Cấu hình Routing Rules (Quy trình chuẩn 2 bước)

Chúng ta sẽ thêm **4 Rules** mới. Mặc định ALB đã có Rule cuối cùng (Default) trỏ về `api-gateway-tg` rồi, không cần sửa.

**Bảng thông số cần nhập:**

| **Priority (Độ ưu tiên)** | **Tên Rule (Name)** | **Condition (Path)** | **Action (Forward to)** |
| --- | --- | --- | --- |
| **1** | Auth Rule | `/api/auth/*` | **`auth-tg`** |
| **2** | User Rule | `/api/user/*` | **`user-tg`** |
| **3** | Task Rule | `/api/taskflow/*` | **`task-tg`** |
| **4** | Noti Rule | `/api/notification/*` | **`noti-tg`** |



Tại tab **Listeners and rules** của ALB, nhấn **Add rule**.

#### Step 1: Define rule (Cấu hình điều kiện & hành động)

1.  **Name and tags:**

    -   **Name:** Nhập tên rule (Ví dụ: `Auth Rule`).

2.  **Conditions:**

    -   Nhấn **Add condition**.

    -   Chọn **Path**.

    -   **Path condition value:** Nhập `/api/auth/*` (Chú ý dấu `*` ở cuối).

    -   Nhấn **Confirm**.

3.  **Actions:**

    -   **Routing action:** Chọn **Forward to target groups**.

    -   **Target group:** Chọn **`auth-tg`**.

    -   **Weight:** `1` (Mặc định).

4.  Kéo xuống dưới cùng, nhấn nút **Next**.

#### Step 2: Set rule policy (Cấu hình độ ưu tiên)

Đây là màn hình bạn đang thấy.

1.  Tại ô **Priority** (bên cạnh tên rule bạn vừa tạo):

    -   Nhập số **`1`** (Vì đây là Auth Rule).

2.  Kiểm tra lại danh sách:

    -   Dòng Priority 1: Auth Rule (Forward to `auth-tg`).

    -   Dòng Last (default): Default (Forward to `api-gateway-tg`).

3.  Nhấn nút **Add rule**.


**Làm tương tự cho 3 rules còn lại:**

1.  **User Rule:**

    -   Step 1: Path `/api/user/*` -> Forward to `user-tg`.

    -   Step 2: Priority nhập **`2`**.

2.  **Task Rule:**

    -   Step 1: Path `/api/taskflow/*` -> Forward to `task-tg`.

    -   Step 2: Priority nhập **`3`**.

3.  **Notification Rule:**

    -   Step 1: Path `/api/notification/*` -> Forward to `noti-tg`.

    -   Step 2: Priority nhập **`4`**.

* * * * *
(Đã đến khúc này lúc 22:26 ngày 05/12/2025)


GIAI ĐOẠN 6: TẠO TASK DEFINITIONS (FINAL VERSION)
=================================================

Mục tiêu: Tạo 6 "bản thiết kế" cho 6 Services để ECS biết cách chạy chúng.

### 1\. Chuẩn bị

Copy sẵn các thông số sau ra Notepad để dùng chung cho tất cả các bước:

1.  **RDS Endpoint:** `sgu-todolist-db.xxxx.ap-southeast-1.rds.amazonaws.com` (Lấy từ RDS).

2.  **Redis Endpoint:** `sgu-redis.xxxx.cache.amazonaws.com` (Lấy từ ElastiCache, **bỏ :6379**).

3.  **Kafka Bootstrap:** `kafka.sgu.local:9092` (Mặc định do cấu hình Service Discovery).

4.  **Google OAuth:** Client ID & Secret (Lấy từ Google Console).

5.  **ECR Image URIs:** Link Image của 6 services (Lấy từ ECR).


### 2\. Tạo Task

Với **mỗi Service** trong danh sách bên dưới, ta thực hiện đúng các bước sau:

1.  Vào **Amazon ECS** > **Task definitions** > **Create new task definition**.

2.  **Task definition family:** Nhập tên (ví dụ: `auth-service-td`).

3.  **Infrastructure requirements:**

    -   Launch type: **AWS Fargate**.

    -   OS Architecture: **Linux/X86_64**.

    -   Task size:

        -   CPU: **`.5 vCPU`**.

        -   Memory: **`1 GB`**.

4.  **Task execution role:** Chọn **Create new role** (hoặc `ecsTaskExecutionRole`).

    -   *Lưu ý: Role này bắt buộc để ECS có quyền kéo Image từ ECR.*

5.  **Container - 1:**

    -   **Name:** Nhập tên (ví dụ: `auth-service`).

    -   **Image URI:** Paste link ECR của service đó.

    -   **Container port:** Nhập Port của service đó (ví dụ: `9999`).

    -   **Protocol:** `TCP`.

    -   **App protocol:** `HTTP`.

6.  **Environment variables:** Nhấn **Add environment variable** để thêm từng dòng theo bảng bên dưới.

7.  **Logging:** Đảm bảo đã tích chọn **Use log collection** (để xem log lỗi trong CloudWatch).

8.  Nhấn **Create**.


### 3\. ChChi tiết cấu hình từng Service

**1. Auth Service**

-   **Family Name:** `auth-service-td`

-   **Name:** auth-service

-   **Container Port:** `9999`

-   **Environment Variables:**

| **Key** | **Value (Thay thế bằng thông tin thật của bạn)** |
| --- | --- |
| `SPRING_DATASOURCE_URL` | `jdbc:mysql://<RDS_ENDPOINT>:3306/aws_todolist_database?allowPublicKeyRetrieval=true&useSSL=false&serverTimezone=UTC` |
| `SPRING_DATASOURCE_USERNAME` | `root` |
| `SPRING_DATASOURCE_PASSWORD` | `12345678` |
| `SPRING_DATA_REDIS_HOST` | `<REDIS_ENDPOINT>` (Không có :6379) |
| `SPRING_DATA_REDIS_PORT` | `6379` |
| `SPRING_KAFKA_BOOTSTRAP_SERVERS` | `kafka.sgu.local:9092` |
| `SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_CLIENT_ID` | `<GOOGLE_CLIENT_ID>` |
| `SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_CLIENT_SECRET` | `<GOOGLE_CLIENT_SECRET>` |
| `SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_REDIRECT_URI` | `https://api.sgutodolist.com/api/auth/login/oauth2/code/google` |
| `APP_OAUTH2_REDIRECT_URI` | `https://sgutodolist.com/oauth2/redirect` |

* * * * *

#### 2\. User Service

-   **Family Name:** `user-service-td`

-   **Container Port:** `8081`

-   **Environment Variables:**

| **Key** | **Value** |
| --- | --- |
| `SPRING_DATASOURCE_URL` | *(Giống Auth Service)* |
| `SPRING_DATASOURCE_USERNAME` | `root` |
| `SPRING_DATASOURCE_PASSWORD` | `12345678` |
| `SPRING_DATA_REDIS_HOST` | *(Giống Auth Service)* |
| `SPRING_DATA_REDIS_PORT` | `6379` |
| `SPRING_KAFKA_BOOTSTRAP_SERVERS` | `kafka.sgu.local:9092` |

* * * * *

#### 3\. Taskflow Service

-   **Family Name:** `taskflow-service-td`

-   **Container Port:** `8082`

-   **Environment Variables:**

| **Key** | **Value** |
| --- | --- |
| `SPRING_DATASOURCE_URL` | *(Giống Auth Service)* |
| `SPRING_DATASOURCE_USERNAME` | `root` |
| `SPRING_DATASOURCE_PASSWORD` | `12345678` |
| `SPRING_DATA_REDIS_HOST` | *(Giống Auth Service)* |
| `SPRING_DATA_REDIS_PORT` | `6379` |
| `SPRING_KAFKA_BOOTSTRAP_SERVERS` | `kafka.sgu.local:9092` |

* * * * *

#### 4\. Notification Service

-   **Family Name:** `notification-service-td`

-   **Container Port:** `9998`

-   **Environment Variables:**

| **Key** | **Value** |
| --- | --- |
| `SPRING_DATASOURCE_URL` | *(Giống Auth Service)* |
| `SPRING_DATASOURCE_USERNAME` | `root` |
| `SPRING_DATASOURCE_PASSWORD` | `12345678` |
| `SPRING_DATA_REDIS_HOST` | *(Giống Auth Service)* |
| `SPRING_DATA_REDIS_PORT` | `6379` |
| `SPRING_KAFKA_BOOTSTRAP_SERVERS` | `kafka.sgu.local:9092` |
| `CLIENT_URL` | `https://sgutodolist.com` |

* * * * *

#### 5\. AI Model Service

-   **Family Name:** `ai-model-service-td`

-   **Container Port:** `9997`

-   **Environment Variables:**

    -   *Nếu code Python Flask của bạn không dùng biến môi trường nào đặc biệt thì để trống phần này.*

    -   *Nếu có kết nối DB, thêm các biến tương tự User Service.*

* * * * *

#### 6\. API Gateway (QUAN TRỌNG)

Service này đóng vai trò "Điều phối viên". Nó cần biết địa chỉ của 5 thằng còn lại.

Chúng ta sẽ sử dụng tên miền nội bộ sgu.local (sẽ được kích hoạt ở Giai đoạn 7).1

-   **Family Name:**  `api-gateway-td`

-   **Container Port:**  `8080`

-   **Environment Variables:**

| **Key** | **Value** | **Giải thích** |
| --- | --- | --- |
| `SPRING_DATA_REDIS_HOST` | `<REDIS_ENDPOINT>` | Rate Limiting |
| `SPRING_DATA_REDIS_PORT` | `6379` |  |
| `AUTH_SERVICE_URL` | `http://auth.sgu.local:9999` | Gọi nội bộ |
| `USER_SERVICE_URL` | `http://user.sgu.local:8081` | Gọi nội bộ |
| `TASKFLOW_SERVICE_URL` | `http://taskflow.sgu.local:8082` | Gọi nội bộ |
| `NOTIFICATION_SERVICE_URL` | `http://notification.sgu.local:9998` | Gọi nội bộ |
| `AI_MODEL_SERVICE` | `http://ai-model.sgu.local:9997` | Gọi nội bộ |
| `CORS_ALLOWED_ORIGINS` | `https://sgutodolist.com` | Cho phép Frontend |

* * * * *

GIAI ĐOẠN 7: DEPLOY SERVICES
----------------------------

Trong mọi bước tạo Service bên dưới, ta **BẮT BUỘC** phải chọn đúng 3 thông số này (để Task tải được Image từ Internet và kết nối được Database):

1.  **VPC:** Chọn `SGU-Microservices-VPC`.

2.  **Subnets:** Phải chọn **2 PUBLIC SUBNETS** (Ví dụ: `...public1...` và `...public2...`).

3.  **Security Group:** Chọn `ecs-app-sg` (Bỏ chọn default).

4.  **Public IP:** **TURNED ON** (Quan trọng nhất).

* * * * *

### NHÓM 1: CÁC SERVICE NGHIỆP VỤ (Auth, User, Taskflow, Notification)

Đây là 4 service vừa cần nhận request từ **Load Balancer** (khi user gọi API), vừa cần **Service Discovery** (để Gateway gọi nội bộ).

Thực hiện quy trình dưới đây **LẶP LẠI 4 LẦN** cho 4 service này.

**Bảng tham số (Dùng để tra cứu khi làm):**

| **Service Cần Tạo** | **Chọn Family (Task Def)** | **Tên Service (ECS)** | **Chọn Target Group (Load Balancer)** | **Tên Service Discovery (Rất quan trọng)** |
| --- | --- | --- | --- | --- |
| **Auth Service** | `auth-service-td` | `auth-service` | `auth-tg` | `auth` |
| **User Service** | `user-service-td` | `user-service` | `user-tg` | `user` |
| **Task Service** | `taskflow-service-td` | `taskflow-service` | `task-tg` | `taskflow` |
| **Noti Service** | `notification-service-td` | `notification-service` | `noti-tg` | `notification` |

**QUY TRÌNH THỰC HIỆN:**

1.  Vào **Clusters** > Chọn Cluster > Tab **Services** > **Create**.

2.  Service details (Thông tin Service)

-   **Task definition family:** Chọn `auth-service-td`.

-   **Task definition revision:** Chọn `Latest`.

-   **Service name:** Nhập `auth-service`.

3. Compute configuration (Cấu hình máy chủ)

-   **Compute options:** Chọn **Capacity provider strategy**.

-   **Capacity provider strategy:** Chọn **Use custom (Advanced)**.

    -   **Capacity provider:** `FARGATE`.

    -   **Base:** `0`.

    -   **Weight:** `1`.

-   **Platform version:** `LATEST`.

4. Deployment configuration (Cấu hình triển khai)

-   **Application type:** Service.

-   **Scheduling strategy:** `Replica`.

-   **Desired tasks:** Nhập `1`.

-   *(Các phần Min/Max running tasks, Deployment circuit breaker...: Để mặc định).*

5. Networking (QUAN TRỌNG)

Đây là phần dễ sai nhất, hãy nhìn kỹ danh sách Subnet bạn gửi:

-   **VPC:** Chọn `SGU-Microservices-vpc`.

-   **Subnets:** Trong danh sách 4 cái hiện ra, bạn **CHỈ CHỌN 2 CÁI CÓ CHỮ "PUBLIC"**:

    1.  ✅ `SGU-Microservices-subnet-public1-ap-southeast-1a`

    2.  ✅ `SGU-Microservices-subnet-public2-ap-southeast-1b`

    -   *(Tuyệt đối KHÔNG chọn private subnets vì bạn không dùng NAT Gateway).*

-   **Security group:**

    -   Bấm dấu `X` để xóa cái `default` đang có sẵn.

    -   Chọn lại group: `ecs-app-sg` (Đã tạo ở Giai đoạn 1).

-   **Public IP:** Phải chọn **Turned on**.

6. Service Connect

-   **Use Service Connect:** **BỎ TRỐNG (Uncheck)**. *(Chúng ta dùng Service Discovery kiểu DNS truyền thống ở bước dưới cho đơn giản).*

7. Service discovery (Cấu hình tên miền nội bộ)

-   Tích chọn ✅ **Use service discovery**.

-   **Config Service Discovery:**

    -   **Select existing namespace:** Chọn `sgu.local`.

    -   **Service discovery name:** Nhập `auth`.

    -   *(Kết quả DNS sẽ là: `auth.sgu.local` - Gateway sẽ dùng tên này để gọi).*

8. Load balancing (Kết nối ALB)

-   Tích chọn ✅ **Use load balancing**.

-   **Load balancer type:** Chọn **Application Load Balancer**.

-   **Load balancer:** Chọn `sgu-alb`.

-   **Container to load balance:** Chọn dòng `auth-service 9999:9999` từ menu thả xuống.

-   **Listener:** Chọn dòng `Use an existing listener` > HTTPS:443.

-   **Target group:**

    -   Chọn **Use an existing target group**.

    -   **Target group name:** Chọn `auth-tg` (Đã tạo ở Giai đoạn 5).

    -   *(Các phần Health check grace period... để mặc định).*

9. Service auto scaling (Tự động mở rộng)

-   **Use service auto scaling:** **BỎ TRỐNG (Uncheck)** (Để tiết kiệm chi phí lúc này).

* * * * *

### NHÓM 2: AI MODEL SERVICE (Nội bộ)

Service này không cần public ra ngoài qua Load Balancer, chỉ cần Gateway gọi nội bộ.

1.  **Family:** `ai-model-service-td`.

2.  **Service name:** `svc-ai-model`.

3.  **Networking:**

    -   VPC: `SGU-Microservices-VPC`.

    -   Subnets: **2 Public Subnets**.

    -   Security group: `ecs-app-sg`.

    -   **Public IP:** **Turned on**.

4.  **Load Balancing:** **KHÔNG CHỌN GÌ CẢ** (Để trống).

5.  **Service Discovery:**

    -   Check vào **Use service discovery**.

    -   Namespace: `sgu.local`.

    -   Service discovery name: `ai-model`.

    -   *(Kết quả DNS: `ai-model.sgu.local`)*.

6.  Click **Create**.

* * * * *

### NHÓM 3: API GATEWAY (Cổng chính)

Service này là cửa ngõ, nó nhận traffic mặc định từ Load Balancer.

1.  **Family:** `api-gateway-td`.

2.  **Service name:** `api-gateway`.

3.  **Networking:**

    -   VPC: `SGU-Microservices-VPC`.

    -   Subnets: **2 Public Subnets**.

    -   Security group: `ecs-app-sg`.

    -   **Public IP:** **Turned on**.

4.  **Load Balancing:**

    -   Load balancer type: **Application Load Balancer**.

    -   Load balancer: `sgu-alb`.

    -   **Container to load balance:** `api-gateway:8080`.

    -   Click **Add to load balancer**.

    -   **Listener:** **Use an existing listener** -> **80:HTTP**.

Sau đó nhấn **Create** là xong nhé! API Gateway sẽ nhận tất cả traffic nào không th

    -   **Target group:** Check vào **Use an existing target group**.

    -   Target group name: **`api-gateway-tg`**.

5.  **Service Discovery:**

    -   Check vào **Use service discovery**.

    -   Namespace: `sgu.local`.

    -   Service discovery name: `api-gateway`.

6.  Click **Create**.

* * * * *

### 🔍 CÁCH KIỂM TRA & XỬ LÝ SỰ CỐ

Sau khi tạo xong cả 6 services, bạn vào tab **Tasks** của Cluster để theo dõi.

1.  **Trạng thái mong muốn:**

    -   Cột **Last status** chuyển từ `PROVISIONING` -> `PENDING` -> `RUNNING` (Màu xanh lá).

    -   Nếu nó đứng ở `PENDING` hơi lâu (1-2 phút) là bình thường (đang tải image).

2.  **Nếu Task bị chuyển sang `STOPPED` (Màu đỏ):**

    -   Đây là chuyện thường gặp lần đầu deploy. Đừng lo lắng.

    -   Click vào mã Task (ví dụ: `e4d3...`).

    -   Nhìn mục **Stopped reason**.

    -   Nếu lý do chung chung, chuyển sang tab **Logs**.

    -   **Các lỗi thường gặp:**

        -   `Connection refused`: Sai thông tin RDS/Redis trong biến môi trường.

        -   `CannotPullContainerError`: Quên bật **Public IP** hoặc Subnet không có Internet.

        -   `Health check failed`: Service khởi động quá lâu (trên 30s) nên ALB tưởng nó chết. (Cách sửa: Vào Target Group > Health checks > Tăng **Interval** lên 60s và **Timeout** lên 30s).

* * * * *

GIAI ĐOẠN 8: HOÀN TẤT
---------------------

1.  **Route 53:** Tạo A Record `api.sgutodolist.com` trỏ về ALB.

2.  **Google Console:** Add Redirect URI `https://api.sgutodolist.com/api/auth/login/oauth2/code/google`.

**CHECKLIST:**

-   Vào `https://api.sgutodolist.com/actuator/health` -> OK.

-   Vào `https://sgutodolist.com` -> Login -> OK.