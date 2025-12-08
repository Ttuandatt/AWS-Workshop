+++
title = "Triển khai Services"
weight = 5
chapter = false
pre = " <b> 5.3.2.5. </b> "
alwaysopen = true
+++

Giai đoạn này triển khai các ứng dụng container lên ECS Fargate theo thứ tự cụ thể để đảm bảo các phụ thuộc được giải quyết đúng cách.

### Cấu hình mạng quan trọng

**Với tất cả các service triển khai, các thiết lập mạng sau là bắt buộc:**

1\. **VPC**: `SGU-Microservices-VPC`

2\. **Subnets**: Chọn cả hai Public Subnets (bắt buộc để kéo image)

3\. **Security Group**: `ecs-app-sg` (xóa default)

4\. **Public IP**: **Bật** (quan trọng cho truy cập ECR)

* * * * *

### Thứ tự triển khai ưu tiên

Các service phải được triển khai theo thứ tự sau để tôn trọng các phụ thuộc:

**Nhóm 1: Business Logic Services** (Triển khai trước - API Gateway cần)  

1\. Auth Service  
2\. User Service  
3\. Taskflow Service  
4\. Notification Service  

**Nhóm 2: Internal Services** (Triển khai thứ hai)  
5\. AI Model Service  

**Nhóm 3: Entry Point** (Triển khai cuối)  
6\. API Gateway  

* * * * *

### Nhóm 1: Triển khai Business Logic Services

Các service này yêu cầu tích hợp ALB (truy cập bên ngoài) và Service Discovery (giao tiếp nội bộ).

**Bảng tham chiếu cấu hình triển khai:**

| Service | Task Definition | Service Name | Target Group | Service Discovery Name |
| --- | --- | --- | --- | --- |
| Auth | `auth-service-td` | `auth-service` | `auth-tg` | `auth` |
| User | `user-service-td` | `user-service` | `user-tg` | `user` |
| Taskflow | `taskflow-service-td` | `taskflow-service` | `task-tg` | `taskflow` |
| Notification | `notification-service-td` | `notification-service` | `noti-tg` | `notification` |

**Quy trình triển khai chuẩn (lặp 4 lần):**

1\. Truy cập **ECS Cluster** → **Services** → **Create**  

2\. **Cấu hình Service:**  
- Task definition family: Chọn task definition tương ứng  
- Revision: Latest  
- Service name: Nhập tên service  

3\. **Cấu hình Compute:**  
- Compute options: Capacity provider strategy  
- Capacity provider strategy: Custom (Advanced)  
  - Capacity provider: FARGATE  
  - Base: 0  
  - Weight: 1  
- Platform version: LATEST  

4\. **Cấu hình Deployment:**  
- Application type: Service  
- Scheduling strategy: Replica  
- Desired tasks: 1  

5\. **Networking:**  
- VPC: `SGU-Microservices-VPC`  
- Subnets: Chọn cả hai Public Subnets  
  - `SGU-Microservices-subnet-public1-ap-southeast-1a`  
  - `SGU-Microservices-subnet-public2-ap-southeast-1b`  
- Security group: Xóa default, chọn `ecs-app-sg`  
- Public IP: Bật  

6\. **Service Connect:** Không bật (sử dụng DNS-based Service Discovery truyền thống)  

7\. **Service Discovery:**  
- Enable: Bật Service Discovery  
- Namespace: `sgu.local`  
- Configure service discovery service: Chọn service discovery hiện có  
- Existing service discovery service: Chọn tên service (ví dụ: `auth`)  
- Result DNS: `auth.sgu.local`  

8\. **Load Balancing:**  
- Enable: Bật Load Balancing  
- Load balancer type: Application Load Balancer  
- Load balancer: `sgu-alb`  
- Container to load balance: Chọn container service (ví dụ: `auth-service 9999:9999`)  
- Listener: Dùng listener hiện có → HTTPS:443  
- Target group: Dùng target group hiện có → Chọn TG tương ứng (ví dụ: `auth-tg`)  

9\. **Service Auto Scaling:** Tắt (tối ưu chi phí)  

10\. Click **Create**  

**Chờ mỗi service đạt trạng thái RUNNING trước khi triển khai service tiếp theo.**

* * * * *

### Nhóm 2: Triển khai AI Model Service

Service nội bộ, chỉ truy cập qua API Gateway.

**Cấu hình:**

1\. Truy cập **ECS Cluster** → **Services** → **Create**  

2\. **Cấu hình Service:**  
- Task definition: `ai-model-service-td`  
- Service name: `ai-model-service`  
- Desired tasks: 1  

3\. **Networking:**  
- VPC: `SGU-Microservices-VPC`  
- Subnets: Cả hai Public Subnets  
- Security group: `ecs-app-sg`  
- Public IP: Bật  

4\. **Load Balancing:** Tắt (internal service)  

5\. **Service Discovery:**  
- Enable: Bật Service Discovery  
- Namespace: `sgu.local`  
- Service discovery name: `ai-model`  
- Result DNS: `ai-model.sgu.local`  

6\. Click **Create**

* * * * *

### Nhóm 3: Triển khai API Gateway

Điểm truy cập chính, điều hướng traffic đến backend services.

**Cấu hình:**

1\. Truy cập **ECS Cluster** → **Services** → **Create**  

2\. **Cấu hình Service:**  
- Task definition: `api-gateway-td`  
- Service name: `api-gateway`  
- Desired tasks: 1  

3\. **Compute Configuration:** Giống Group 1  

4\. **Networking:**  
- VPC: `SGU-Microservices-VPC`  
- Subnets: Cả hai Public Subnets  
- Security group: `ecs-app-sg`  
- Public IP: Bật  

5\. **Service Discovery:**  
- Enable: Bật Service Discovery  
- Namespace: `sgu.local`  
- Service discovery name: `api-gateway`  

6\. **Load Balancing:**  
- Enable: Bật Load Balancing  
- Load balancer type: Application Load Balancer  
- Load balancer: `sgu-alb`  
- Container to load balance: `api-gateway:8080`  
- Listener: HTTP:80  
- Target group: `api-gateway-tg`  

7\. Click **Create**

* * * * *

### Giám sát triển khai và xử lý sự cố

Sau khi triển khai tất cả service, theo dõi trạng thái trong ECS Cluster.

**Trạng thái Task dự kiến:**

- PROVISIONING → PENDING → RUNNING  
- Trạng thái PENDING: bình thường 1-2 phút (download image từ ECR)  
- RUNNING: Task hoạt động, phục vụ traffic  

**Vấn đề phổ biến và cách xử lý:**

#### Vấn đề 1: Task = STOPPED

**Triệu chứng**: Task dừng ngay sau khi khởi động  

**Chẩn đoán:**  
1\. Click vào task ID bị stop  
2\. Xem **Stopped reason**  
3\. Chuyển sang tab **Logs** để xem lỗi chi tiết  

**Nguyên nhân phổ biến:**

| Error Message | Nguyên nhân | Giải pháp |
| --- | --- | --- |
| `Connection refused` | Endpoint RDS/Redis sai trong biến môi trường | Kiểm tra lại giá trị endpoint trong Task Definition |
| `CannotPullContainerError` | Public IP tắt hoặc subnet không có internet | Bật Public IP và dùng Public Subnets |
| `Health check failed` | Thời gian khởi động service vượt quá timeout của ALB | Tăng health check interval và timeout trong Target Group |
| `ResourceInitializationError` | CPU/Memory không đủ | Kiểm tra cấu hình task size |

**Giải pháp cho Health Check Failures:**  
- Vào **Target Group** → Chọn TG gặp lỗi  
- Tab **Health checks** → Edit  
- Tăng giá trị: Interval = 60s, Timeout = 30s, Healthy threshold = 3  
- Save changes  

#### Vấn đề 2: Container liên tục Restart

**Nguyên nhân có thể:**  
- Kết nối DB thất bại  
- Biến môi trường thiếu  
- Cấu hình ứng dụng sai  

**Chẩn đoán:**  
1\. Kiểm tra CloudWatch Logs  
2\. Xác nhận tất cả biến môi trường đúng  
3\. Kiểm tra RDS và Redis có truy cập được từ ECS security group không  

* * * * *

### Checklist xác minh triển khai

**Trạng thái Services:**

- [ ] Auth Service: RUNNING (1/1 tasks)  
- [ ] User Service: RUNNING (1/1 tasks)  
- [ ] Taskflow Service: RUNNING (1/1 tasks)  
- [ ] Notification Service: RUNNING (1/1 tasks)  
- [ ] AI Model Service: RUNNING (1/1 tasks)  
- [ ] API Gateway: RUNNING (1/1 tasks)  

**Target Groups Health:**

- [ ] `auth-tg`: 1 healthy target  
- [ ] `user-tg`: 1 healthy target  
- [ ] `task-tg`: 1 healthy target  
- [ ] `noti-tg`: 1 healthy target  
- [ ] `api-gateway-tg`: 1 healthy target  

**Service Discovery:**

- [ ] Tất cả services đã đăng ký trong Cloud Map  
- [ ] DNS names resolve chính xác (kiểm tra trong Route 53)  

* * * * *

<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.4-Task Definitions Creation (Configure settings, FIX environment variables)" %}}" style="text-decoration: none; font-weight: bold;">
⬅ BƯỚC 4: Task Definitions Creation
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.6-Completion & Verification (Route 53, Google Console, Final Test)" %}}" style="text-decoration: none; font-weight: bold;">
BƯỚC 6: Completion & Verification ➡
</a>
</div>
