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

1.  Vào ECS > Task Definitions > Create.

2.  Family: `kafka-server-td`.

3.  Container:

    -   Name: `kafka-server`

    -   Image: `bitnami/kafka:latest`

    -   Port: `9092`

    -   **Environment Variables:**

        -   `KAFKA_CFG_NODE_ID` = `0`

        -   `KAFKA_CFG_PROCESS_ROLES` = `controller,broker`

        -   `KAFKA_CFG_LISTENERS` = `PLAINTEXT://:9092,CONTROLLER://:9093`

        -   `KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP` = `CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT`

        -   `KAFKA_CFG_CONTROLLER_QUORUM_VOTERS` = `0@kafka.sgu.local:9093`

        -   `KAFKA_CFG_ADVERTISED_LISTENERS` = `PLAINTEXT://kafka.sgu.local:9092`

4.  Create.

**Bước 3: Deploy Kafka Service**

1.  ECS > Cluster > Services > Create.

2.  Family: `kafka-server-td`. Service name: `svc-kafka`.

3.  **Networking:**

    -   Subnets: **2 Public Subnets** (Bắt buộc vì không có NAT).

    -   Security Group: `private-db-sg`.

    -   Public IP: **Turned ON**.

4.  **Service Connect and Service Discovery:**

    -   Use service discovery: **Check**.

    -   Namespace: `sgu.local`.

    -   Service name: `kafka`. (Kết quả DNS: `kafka.sgu.local`).

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


**👉 Làm tương tự cho 3 rules còn lại:**

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


GIAI ĐOẠN 6: TASK DEFINITIONS (KHỚP DOCKER-COMPOSE)
---------------------------------------------------

### 1\. Auth Service & User Service & Taskflow Service

*(Tạo 3 Task riêng, cấu hình giống nhau, khác Image và Port)*

-   **Ports:** Auth (9999), User (8081), Taskflow (8082).

-   **Env Vars (Copy chính xác):**

| **Key** | **Value** |
| --- | --- |
| `SPRING_DATASOURCE_URL` | `jdbc:mysql://<RDS-ENDPOINT>:3306/aws_todolist_database?allowPublicKeyRetrieval=true&useSSL=false&serverTimezone=UTC` |
| `SPRING_DATASOURCE_USERNAME` | `root` |
| `SPRING_DATASOURCE_PASSWORD` | `12345678` |
| `SPRING_DATA_REDIS_HOST` | `<REDIS-ENDPOINT>` |
| `SPRING_DATA_REDIS_PORT` | `6379` |
| `SPRING_KAFKA_BOOTSTRAP_SERVERS` | `kafka.sgu.local:9092` |
| `SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_REDIRECT_URI` | `https://api.sgutodolist.com/api/auth/login/oauth2/code/google` |
| `APP_OAUTH2_REDIRECT_URI` | `https://sgutodolist.com/oauth2/redirect` |

*(Lưu ý: User & Taskflow Service không cần các biến OAUTH2, chỉ Auth cần, nhưng để chung cũng không lỗi).*

* * * * *

### 2\. Notification Service

-   **Port:** 9998.

-   **Env Vars:** Giống trên, thêm biến `CLIENT_URL`: `https://sgutodolist.com`.

* * * * *

### 3\. AI Model Service (MỚI)

-   **Family:** `ai-model-service-td`.

-   **Image:** Link ECR AI Model.

-   **Port:** 9997.

-   **Env Vars:** (Tùy theo code Flask của bạn, nếu không có thì để trống).

* * * * *

### 4\. API Gateway (QUAN TRỌNG)

-   **Port:** 8080.

-   **Env Vars:**

| **Key** | **Value** |
| --- | --- |
| `SPRING_DATA_REDIS_HOST` | `<REDIS-ENDPOINT>` |
| `SPRING_DATA_REDIS_PORT` | `6379` |
| `AUTH_SERVICE_URL` | `https://api.sgutodolist.com/api/auth` |
| `USER_SERVICE_URL` | `https://api.sgutodolist.com/api/users` |
| `TASKFLOW_SERVICE_URL` | `https://api.sgutodolist.com/api/tasks` |
| `NOTIFICATION_SERVICE_URL` | `https://api.sgutodolist.com/api/notifications` |
| `AI_MODEL_SERVICE` | `http://ai-model.sgu.local:9997` |
| `CORS_ALLOWED_ORIGINS` | `https://sgutodolist.com,https://www.sgutodolist.com` |

* * * * *

GIAI ĐOẠN 7: DEPLOY SERVICES
----------------------------

Deploy lần lượt 6 Services (Auth, User, Taskflow, Notification, Gateway, AI Model).

**CẤU HÌNH CHUNG KHI TẠO SERVICE:**

1.  **VPC:** `SGU-Microservices-VPC`.

2.  **Subnets:** **2 PUBLIC SUBNETS**. (Bắt buộc vì không có NAT).

3.  **Security Group:** `ecs-app-sg`.

4.  **Public IP:** **TURN ON** (Bắt buộc).

**RIÊNG CHO AI MODEL SERVICE:**

-   Khi tạo service `svc-ai-model`:

-   **Service discovery:** Check.

-   Namespace: `sgu.local`.

-   Service name: `ai-model`.

-   *(Để Gateway gọi được qua `http://ai-model.sgu.local:9997`).*

* * * * *

GIAI ĐOẠN 8: HOÀN TẤT
---------------------

1.  **Route 53:** Tạo A Record `api.sgutodolist.com` trỏ về ALB.

2.  **Google Console:** Add Redirect URI `https://api.sgutodolist.com/api/auth/login/oauth2/code/google`.

**CHECKLIST:**

-   Vào `https://api.sgutodolist.com/actuator/health` -> OK.

-   Vào `https://sgutodolist.com` -> Login -> OK.