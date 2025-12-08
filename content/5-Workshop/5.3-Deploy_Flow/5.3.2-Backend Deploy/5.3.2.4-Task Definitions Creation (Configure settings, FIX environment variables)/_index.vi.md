+++
title = "Tạo Task Definitions"
weight = 4
chapter = false
pre = " <b> 5.3.2.4. </b> "
alwaysopen = true
+++

Task Definitions đóng vai trò như bản thiết kế, xác định cách ECS chạy các container, bao gồm phân bổ tài nguyên (RAM/CPU) và các biến môi trường quan trọng để kết nối giữa các service.

### Chuẩn bị

**Ghi lại các giá trị sau trước khi bắt đầu:**

1. **RDS Endpoint**: `sgu-todolist-db.[random].ap-southeast-1.rds.amazonaws.com`

2. **Redis Endpoint**: `sgu-redis.[random].cache.amazonaws.com` (không kèm :6379)

3. **ECR Image URIs**: URIs của 6 repository từ bước Xây Dựng Image.

4. **Google OAuth Credentials**: Client ID và Client Secret.

* * * * *

### Quy trình chuẩn tạo Task Definition

Với mỗi service, làm theo quy trình cấu hình chuẩn sau:

**1\. Cấu hình cơ bản:**

- Truy cập **Amazon ECS** → **Task definitions** → **Create new task definition**.

- **Hạ tầng:**

    - Launch type: **AWS Fargate**  
    - Operating system: **Linux/X86_64**  
    - Task Execution Role: `ecsTaskExecutionRole`  

- **Chi tiết Container:**

    - Protocol: **TCP**  
    - Port Mapping: Tham khảo chi tiết từng service bên dưới.  
    - Log collection: **Bật** (Quan trọng cho debug).  

* * * * *

### 1\. Kafka Server (Hạ tầng nền tảng)

*Lưu ý: Kafka phải được triển khai trước để các service khác có thể kết nối khi khởi động.*

| **Tham số** | **Giá trị** |
| --- | --- |
| **Family Name** | `kafka-server-td` |
| **Task Memory** | **2 GB** (Quan trọng) |
| **Task CPU** | **1 vCPU** |
| **Container Port** | `9092` |

**Biến môi trường (copy chính xác):**

| **Key** | **Value** |
| --- | --- |
| `ALLOW_PLAINTEXT_LISTENER` | `yes` |
| `KAFKA_CFG_ADVERTISED_LISTENERS` | `PLAINTEXT://kafka.sgu.local:9092` |
| `KAFKA_CFG_CONTROLLER_LISTENER_NAMES` | `CONTROLLER` |
| `KAFKA_CFG_CONTROLLER_QUORUM_VOTERS` | `0@127.0.0.1:9093` |
| `KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP` | `CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT` |
| `KAFKA_CFG_LISTENERS` | `PLAINTEXT://:9092,CONTROLLER://:9093` |
| `KAFKA_CFG_LOG_DIRS` | `/tmp/kafka-logs` |
| `KAFKA_CFG_NODE_ID` | `0` |
| `KAFKA_CFG_PROCESS_ROLES` | `controller,broker` |
| `KAFKA_HEAP_OPTS` | `-Xmx512m` |

* * * * *

### 2\. Auth Service (Service bảo mật quan trọng)

*Lưu ý: Service này cần RAM cao để xử lý Login và Encryption.*

| **Tham số** | **Giá trị** |
| --- | --- |
| **Family Name** | `auth-service-td` |
| **Task Memory** | **2 GB** (Bắt buộc tránh Exit Code 137) |
| **Task CPU** | **0.5 vCPU** |
| **Container Port** | `9999` |

**Biến môi trường:**

| **Key** | **Value** | **Ghi chú** |
| --- | --- | --- |
| `SERVER_PORT` | `9999` |  |
| `JAVA_OPTS` | `-Xmx768m` | Phải bao gồm dấu `-`. |
| `SPRING_JPA_HIBERNATE_DDL_AUTO` | **`none`** | **CRITICAL:** Ngăn lỗi 500 do xung đột schema DB. |
| `SERVER_FORWARD_HEADERS_STRATEGY` | `native` | Khắc phục lỗi redirect HTTP/HTTPS phía sau ALB. |
| `SPRING_DATASOURCE_URL` | `jdbc:mysql://[RDS-ENDPOINT]:3306/aws_todolist_database?allowPublicKeyRetrieval=true&useSSL=false&serverTimezone=UTC` |  |
| `SPRING_DATASOURCE_USERNAME` | `root` |  |
| `SPRING_DATASOURCE_PASSWORD` | `[DB_PASSWORD]` |  |
| `SPRING_DATASOURCE_HIKARI_MAXIMUM_POOL_SIZE` | `5` |  |
| `SPRING_DATA_REDIS_HOST` | `[REDIS-ENDPOINT]` |  |
| `SPRING_DATA_REDIS_PORT` | `6379` |  |
| `SPRING_DATA_REDIS_SSL_ENABLED` | **`true`** | Bắt buộc cho AWS ElastiCache. |
| `SPRING_KAFKA_BOOTSTRAP_SERVERS` | `kafka.sgu.local:9092` |  |
| `DOMAIN_FRONTEND` | `https://sgutodolist.com` |  |
| `APP_OAUTH2_REDIRECT_URI` | `https://sgutodolist.com/oauth2/redirect` |  |
| `SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_CLIENT_ID` | `[GOOGLE_CLIENT_ID]` |  |
| `SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_CLIENT_SECRET` | `[GOOGLE_CLIENT_SECRET]` |  |
| `SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_REDIRECT_URI` | `https://sgutodolist.com/api/auth/login/oauth2/code/google` | Lưu ý: Bao gồm `/api/auth` trong đường dẫn. |

* * * * *

### 3\. User Service & Taskflow Service

*Đây là các service xử lý business logic, cần quyền tạo bảng DB khi chạy lần đầu.*

| **Tham số** | **Giá trị** |
| --- | --- |
| **Family Name** | `user-service-td` / `taskflow-service-td` |
| **Task Memory** | **1 GB** |
| **Task CPU** | **0.5 vCPU** |
| **Container Port** | `8081` (User) / `8082` (Taskflow) |

**Biến môi trường (Chung cho cả hai):**

| **Key** | **Value** | **Ghi chú** |
| --- | --- | --- |
| `SERVER_PORT` | `8081` (User) / `8082` (Taskflow) |  |
| `JAVA_OPTS` | `-Xmx512m` |  |
| `SPRING_JPA_HIBERNATE_DDL_AUTO` | **`update`** | Cho phép service tự tạo bảng khi khởi động. |
| `SPRING_DATASOURCE_URL` | *(Giống Auth Service)* |  |
| `SPRING_DATASOURCE_USERNAME` | `root` |  |
| `SPRING_DATASOURCE_PASSWORD` | `[DB_PASSWORD]` |  |
| `SPRING_DATASOURCE_HIKARI_MAXIMUM_POOL_SIZE` | `5` |  |
| `SPRING_DATA_REDIS_HOST` | `[REDIS-ENDPOINT]` |  |
| `SPRING_DATA_REDIS_PORT` | `6379` |  |
| `SPRING_DATA_REDIS_SSL_ENABLED` | `true` |  |
| `SPRING_KAFKA_BOOTSTRAP_SERVERS` | `kafka.sgu.local:9092` |  |

* * * * *

### 4\. API Gateway (Orchestrator)

*Service quan trọng nhất cho routing nội bộ.*

| **Tham số** | **Giá trị** |
| --- | --- |
| **Family Name** | `api-gateway-td` |
| **Task Memory** | **1 GB** |
| **Task CPU** | **0.5 vCPU** |
| **Container Port** | `8080` |

**Biến môi trường:**

| **Key** | **Value** | **Giải thích** |
| --- | --- | --- |
| `SERVER_PORT` | `8080` |  |
| `CORS_ALLOWED_ORIGINS` | `https://sgutodolist.com,https://www.sgutodolist.com` | Cho phép Frontend truy cập API. |
| `AUTH_SERVICE_URL` | `http://auth.sgu.local:9999` | Routing nội bộ qua Service Discovery. |
| `USER_SERVICE_URL` | **`http://user.sgu.local:8081`** | Lưu ý tên: `user` (không phải `user-service`). |
| `TASKFLOW_SERVICE_URL` | **`http://taskflow.sgu.local:8082`** | Lưu ý tên: `taskflow`. |
| `NOTIFICATION_SERVICE_URL` | **`http://notification.sgu.local:9998`** | Lưu ý tên: `notification`. |
| `SPRING_DATA_REDIS_HOST` | `[REDIS-ENDPOINT]` | Dùng cho Rate Limiting. |
| `SPRING_DATA_REDIS_PORT` | `6379` |  |
| `SPRING_DATA_REDIS_SSL_ENABLED` | `true` |  |

* * * * *

### Checklist xác minh Task Definition

Sau khi tạo xong, kiểm tra:

- [ ] **RAM Allocation:** Auth Service phải **2GB**, các service khác ít nhất **1GB**.  
- [ ] **Redis SSL:** Biến `SPRING_DATA_REDIS_SSL_ENABLED` = `true` được set cho tất cả service kết nối Redis.  
- [ ] **Internal URLs:** Tất cả `_SERVICE_URL` trong Gateway phải kết thúc với `.sgu.local`.  
- [ ] **DDL Auto:** Auth Service = `none`, User/Taskflow = `update`.  

<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.3-Code Update & Image Build (Create new Docker Image)" %}}" style="text-decoration: none; font-weight: bold;">
⬅ BƯỚC 3: Cập Nhật Code & Xây Dựng Image
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.5-Services Deployment" %}}" style="text-decoration: none; font-weight: bold;">
BƯỚC 5: Triển khai Services ➡
</a>
</div>
