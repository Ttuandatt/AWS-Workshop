+++
title = "Thiết lập hạ tầng &ALB (RDS, Redis, Cloud Map, ALB Routing)"
weight = 2
chapter = false
pre = " <b> 5.3.2.2. </b> "
alwaysopen = true
+++


Giai đoạn này triển khai các thành phần tầng dữ liệu, cơ chế service discovery và hạ tầng cân bằng tải.

### Cấp phát SSL Certificate

**Bước 1: Yêu cầu ACM Certificate**

1.  Chuyển region sang **Singapore (ap-southeast-1)**
2.  Điều hướng đến **Certificate Manager (ACM)**
3.  Chọn **Request a certificate** → **Request a public certificate**
4.  Cấu hình yêu cầu certificate:
    -   **Domain name**: `sgutodolist.com`
    -   **Validation method**: DNS validation
5.  Nhấn **Request**
6.  Truy cập chi tiết certificate → Trong phần **Domains** → Nhấn **Create records in Route 53**
7.  Đợi trạng thái chuyển thành **Issued** (thường mất 5-10 phút)

* * * * *

### Hạ tầng Tầng Dữ liệu

#### RDS MySQL Database

**Mục đích**: Cơ sở dữ liệu quan hệ chính cho tất cả các microservices.

**Các bước cấu hình:**

1.  Điều hướng đến **RDS Console** → **Create database**
2.  Chọn **Standard create** → Engine **MySQL**
3.  Cấu hình database instance:

| Tham số | Giá trị | Lý do |
| --- | --- | --- |
| Template | Free tier | Tối ưu chi phí |
| DB instance identifier | `sgu-todolist-db` | Quy ước đặt tên |
| Master username | `root` | Tài khoản admin chuẩn |
| Master password | `[mật-khẩu-bảo-mật]` | Ghi chép cẩn thận |
| DB instance class | `db.t3.micro` | Đủ điều kiện free tier |
| Allocated storage | 20 GiB | Giới hạn free tier |

1.  Cấu hình mạng:
    -   **Compute resource**: Don't connect to an EC2 compute resource
    -   **VPC**: `SGU-Microservices-VPC`
    -   **Public access**: No
    -   **VPC security group**: Chọn `private-db-sg` (xóa default)
    -   **Availability Zone**: `ap-southeast-1a`
2.  Nhấn **Create database**
3.  Sau khi tạo xong, ghi lại địa chỉ **Endpoint** (định dạng: `sgu-todolist-db.[random].ap-southeast-1.rds.amazonaws.com`)

* * * * *

#### ElastiCache Redis

**Mục đích**: Tầng caching cho quản lý session, rate limiting và lưu trữ dữ liệu tạm thời.

**Các bước cấu hình:**

**Giai đoạn 1 - Cài đặt cơ bản:**

1.  Điều hướng đến **ElastiCache Console** → **Redis OSS caches** → **Create Redis OSS cache**
2.  Cấu hình cluster:

| Tham số | Giá trị | Lý do |
| --- | --- | --- |
| Engine | Redis OSS | Tương thích open source |
| Deployment option | Node-based cluster | Triển khai single node |
| Creation method | Cluster cache | Tạo chuẩn |
| Cluster mode | Disabled | Kiến trúc đơn giản |
| Cluster name | `sgu-redis` | Quy ước đặt tên |
| Multi-AZ | Disabled | Tối ưu chi phí |
| Auto-failover | Disabled | Không cần cho môi trường phát triển |
| Node type | `cache.t3.micro` | Đủ điều kiện free tier |
| Number of replicas | 0 | Cấu hình single node |

1.  Cấu hình mạng:
    -   **Subnet groups**: Create a new subnet group
        -   **Name**: `sgu-redis-subnet-group`
        -   **VPC**: `SGU-Microservices-VPC`
        -   **Selected subnets**: Cả hai Private Subnets
    -   **Availability Zone placement**: Specify Availability Zones
        -   **Primary**: `ap-southeast-1a`

**Giai đoạn 2 - Cài đặt nâng cao:**

1.  Cấu hình bảo mật:
    -   **Security groups**: Chọn `private-db-sg` (xóa default)
2.  Cấu hình backup:
    -   **Enable automatic backups**: Disabled
3.  Logs: Tắt tất cả các loại log
4.  Nhấn **Create**
5.  Ghi lại **Primary Endpoint** (định dạng: `sgu-redis.[random].cache.amazonaws.com`)

**Quan trọng:** Không bao gồm `:6379` khi ghi endpoint cho biến môi trường.

* * * * *

#### Thiết lập Service Discovery

**Mục đích**: Phân giải DNS nội bộ cho giao tiếp giữa các microservices.

**Bước 1: Tạo Cloud Map Namespace**

1.  Điều hướng đến **AWS Cloud Map** → **Create namespace**
2.  Cấu hình namespace:
    -   **Name**: `sgu.local`
    -   **Instance discovery**: API calls and DNS queries in VPCs
    -   **VPC**: `SGU-Microservices-VPC`
3.  Nhấn **Create**

**Kết quả**: Tên miền DNS nội bộ `sgu.local` hiện đã sẵn sàng để đăng ký dịch vụ.

* * * * *

#### Apache Kafka Message Broker

**Mục đích**: Event streaming và giao tiếp bất đồng bộ giữa các microservices.

**Bước 1: Tạo Kafka Task Definition**

1.  Điều hướng đến **Amazon ECS** → **Task definitions** → **Create new task definition**
2.  Cấu hình hạ tầng:

| Tham số | Giá trị |
| --- | --- |
| Task definition family | `kafka-server-td` |
| Launch type | AWS Fargate |
| Operating system | Linux/x86_64 |
| CPU | 0.5 vCPU |
| Memory | 1 GB |
| Task execution role | `ecsTaskExecutionRole` |

1.  Cấu hình container:
    -   **Name**: `kafka-server`
    -   **Image URI**: `bitnami/kafka:latest`
    -   **Port mappings**:
        -   Container port: `9092`
        -   Protocol: TCP
2.  Biến môi trường:

| Key | Value | Mục đích |
| --- | --- | --- |
| `KAFKA_CFG_NODE_ID` | `0` | Định danh node |
| `KAFKA_CFG_PROCESS_ROLES` | `controller,broker` | Chế độ kết hợp (KRaft) |
| `KAFKA_CFG_LISTENERS` | `PLAINTEXT://:9092,CONTROLLER://:9093` | Listeners nội bộ |
| `KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP` | `CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT` | Ánh xạ protocol |
| `KAFKA_CFG_CONTROLLER_QUORUM_VOTERS` | `0@127.0.0.1:9093` | Tự tham chiếu |
| `KAFKA_CFG_ADVERTISED_LISTENERS` | `PLAINTEXT://kafka.sgu.local:9092` | Địa chỉ service discovery |
| `KAFKA_CFG_LOG_DIRS` | `/tmp/kafka-logs` | Lưu trữ log tạm thời |
| `ALLOW_PLAINTEXT_LISTENER` | `yes` | Giao tiếp không SSL |

1.  Bật **Use log collection** cho CloudWatch Logs
2.  Nhấn **Create**

**Bước 2: Triển khai Kafka Service**

1.  Điều hướng đến ECS Cluster → **Services** → **Create**
2.  Cấu hình service:
    -   **Task definition**: `kafka-server-td` (phiên bản mới nhất)
    -   **Service name**: `kafka-server`
    -   **Desired tasks**: 1
3.  Cấu hình mạng:
    -   **VPC**: `SGU-Microservices-VPC`
    -   **Subnets**: Cả hai Public Subnets
    -   **Security group**: `private-db-sg`
    -   **Public IP**: Enabled
4.  Load balancing: Không cấu hình
5.  Service Discovery:
    -   Bật **Use service discovery**
    -   **Namespace**: `sgu.local`
    -   **Service discovery name**: `kafka`
    -   **Result DNS**: `kafka.sgu.local:9092`
6.  Nhấn **Create**

* * * * *

#### Khởi tạo Database

**Mục đích**: Tạo schema database ứng dụng một cách an toàn thông qua jump server.

**Bước 1: Cập nhật Security Group**

1.  Điều hướng đến `private-db-sg` → **Inbound Rules** → **Edit**
2.  Thêm rule tạm thời:
    -   **Type**: MySQL/Aurora (port 3306)
    -   **Source**: `bastion-sg`
3.  Lưu rules

**Bước 2: Tạo Bastion Host**

1.  Điều hướng đến **EC2** → **Launch Instance**
2.  Cấu hình instance:

| Tham số | Giá trị |
| --- | --- |
| Name | `sgu-bastion` |
| AMI | Amazon Linux 2023 |
| Instance type | `t3.micro` |
| Key pair | Tạo mới: `sgutodolist-key` (tải .pem) |
| VPC | `SGU-Microservices-VPC` |
| Subnet | Public Subnet 1 |
| Auto-assign public IP | Enable |
| Security group | `bastion-sg` |

1.  Nhấn **Launch instance**
2.  Ghi lại địa chỉ **Public IPv4 address**

**Bước 3: Kết nối và Khởi tạo Database**

**Windows (PowerShell):**

powershell

```
# Thiết lập quyền chính xác
icacls "sgutodolist-key.pem" /inheritance:r
icacls "sgutodolist-key.pem" /grant:r "$($env:USERNAME):(R)"

# Kết nối đến Bastion
ssh -i "sgutodolist-key.pem" ec2-user@[BASTION-PUBLIC-IP]
```

**macOS/Linux:**

bash

```
# Thiết lập quyền chính xác
chmod 400 sgutodolist-key.pem

# Kết nối đến Bastion
ssh -i sgutodolist-key.pem ec2-user@[BASTION-PUBLIC-IP]
```

**Trên Bastion Host:**

bash

```
# Cài đặt MySQL client
sudo dnf install mariadb105 -y

# Kết nối đến RDS
mysql -h [RDS-ENDPOINT] -u root -p
# Nhập password khi được yêu cầu

# Tạo database
CREATE DATABASE aws_todolist_database
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;

# Kiểm tra
SHOW DATABASES;

# Thoát
EXIT;
exit
```

**Bước 4: Dọn dẹp Bảo mật**

Xóa quyền truy cập database tạm thời:

1.  Quay lại `private-db-sg` → **Inbound Rules**
2.  Xóa rule cho phép `bastion-sg` truy cập port 3306

* * * * *

### Cấu hình Application Load Balancer

#### Tạo Target Groups

Tạo 5 target groups cho các microservices sẽ nhận traffic bên ngoài.

**Quy trình Cấu hình Chuẩn (Lặp lại 5 lần):**

1.  Điều hướng đến **EC2** → **Target Groups** → **Create target group**
2.  Cấu hình cơ bản:
    -   **Target type**: IP addresses (bắt buộc cho Fargate)
    -   **Protocol**: HTTP
    -   **IP address type**: IPv4
    -   **VPC**: `SGU-Microservices-VPC`
    -   **Protocol version**: HTTP1
3.  Cấu hình health check:
    -   **Health check protocol**: HTTP
    -   **Health check path**: `/actuator/health`
4.  Nhấn **Next** (không đăng ký targets thủ công)
5.  Nhấn **Create target group**

**Chi tiết Target Groups:**

| # | Tên Target Group | Port | Health Check Path | Mục đích |
| --- | --- | --- | --- | --- |
| 1 | `api-gateway-tg` | 8080 | `/actuator/health` | API Gateway |
| 2 | `auth-tg` | 9999 | `/actuator/health` | Auth Service |
| 3 | `user-tg` | 8081 | `/actuator/health` | User Service |
| 4 | `task-tg` | 8082 | `/actuator/health` | Taskflow Service |
| 5 | `noti-tg` | 9998 | `/actuator/health` | Notification Service |

**Lưu ý:** AI Model Service không cần target group vì chỉ được truy cập nội bộ qua API Gateway.

* * * * *

#### Tạo Load Balancer

**Bước 1: Tạo ALB**

1.  Điều hướng đến **EC2** → **Load Balancers** → **Create load balancer**
2.  Chọn **Application Load Balancer**
3.  Cấu hình cơ bản:

| Tham số | Giá trị |
| --- | --- |
| Name | `sgu-alb` |
| Scheme | Internet-facing |
| IP address type | IPv4 |

1.  Ánh xạ mạng:
    -   **VPC**: `SGU-Microservices-VPC`
    -   **Availability Zones**: Chọn cả hai
        -   `ap-southeast-1a` → Public Subnet 1
        -   `ap-southeast-1b` → Public Subnet 2
2.  Security groups:
    -   Xóa `default`
    -   Chọn `public-alb-sg`
3.  Listeners và routing:
    -   **Listener 1 (HTTP:80)**:
        -   Protocol: HTTP | Port: 80
        -   Default action: Forward to `api-gateway-tg`
    -   **Listener 2 (HTTPS:443)**: Nhấn **Add listener**
        -   Protocol: HTTPS | Port: 443
        -   Default action: Forward to `api-gateway-tg`
        -   Certificate source: ACM
        -   Certificate: Chọn `sgutodolist.com`
4.  Nhấn **Create load balancer**
5.  Đợi trạng thái chuyển từ `Provisioning` sang `Active`

* * * * *

#### Cấu hình Routing Rules

Cấu hình định tuyến dựa trên path cho HTTPS listener.

**Bước 1: Truy cập Listener Rules**

1.  Chọn ALB → Tab **Listeners and rules**
2.  Chọn listener **HTTPS:443** → Nhấn **Manage rules**

**Bước 2: Thêm Routing Rules**

Tạo 4 rules với các thông số sau:

| Ưu tiên | Tên Rule | Path Pattern | Target Group |
| --- | --- | --- | --- |
| 1 | Auth Rule | `/api/auth/*` | `auth-tg` |
| 2 | User Rule | `/api/user/*` | `user-tg` |
| 3 | Task Rule | `/api/taskflow/*` | `task-tg` |
| 4 | Noti Rule | `/api/notification/*` | `noti-tg` |

**Quy trình Tạo Rule (Lặp lại 4 lần):**

1.  Nhấn **Add rule**
2.  **Bước 1 - Define rule**:
    -   **Name**: Nhập tên rule (ví dụ: "Auth Rule")
    -   **Conditions**: Add condition → Path → Nhập path pattern (ví dụ: `/api/auth/*`)
    -   **Actions**: Forward to target groups → Chọn target group tương ứng
    -   Nhấn **Next**
3.  **Bước 2 - Set rule priority**:
    -   **Priority**: Nhập số ưu tiên (1-4)
    -   Nhấn **Add rule**

**Kiểm tra**: Rule mặc định chuyển tiếp đến `api-gateway-tg` nên ở cuối với ưu tiên "Last".

* * * * *

### Xác thực Hạ tầng

Trước khi tiến hành triển khai code, kiểm tra:

**Tầng Dữ liệu:**

-   [ ]  Trạng thái RDS instance: Available
-   [ ]  Đã ghi endpoint RDS
-   [ ]  Database `aws_todolist_database` đã được tạo

**Tầng Caching:**

-   [ ]  Trạng thái Redis cluster: Available
-   [ ]  Đã ghi endpoint Redis (không có :6379)

**Service Discovery:**

-   [ ]  Cloud Map namespace `sgu.local` đã được tạo
-   [ ]  Kafka service đang chạy với DNS `kafka.sgu.local`

**Load Balancer:**

-   [ ]  Trạng thái ALB: Active
-   [ ]  5 target groups đã được tạo
-   [ ]  HTTPS listener đã cấu hình với ACM certificate
-   [ ]  4 path-based routing rules đã được cấu hình

* * * * *

<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.1-Network & Security Preparation (VPC, SG)" %}}" style="text-decoration: none; font-weight: bold;">
⬅ BƯỚC 1: Chuẩn bị Mạng & Bảo mật
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.3-Code Update & Image Build (Create new Docker Image)" %}}" style="text-decoration: none; font-weight: bold;">
BƯỚC 3: Cập Nhật Code & Xây Dựng Image ➡
</a>
</div>