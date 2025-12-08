+++
title = "Tổng quan Workshop"
weight = 1
chapter = false
pre = " <b> 5.1.  </b> "
+++

Giới thiệu dự án
-----------------

**SGU TodoList** là một ứng dụng quản lý công việc (Task Management) được xây dựng theo kiến trúc **Microservices** trên nền tảng **AWS Cloud**.  
Dự án ban đầu được thiết kế theo mô hình **Multi-Region SaaS** nhằm đảm bảo **High Availability** và **Disaster Recovery**. Tuy nhiên, do hạn chế về ngân sách và giới hạn của **AWS Free Tier**, nhóm đã tối ưu kiến trúc sang mô hình **Single-Region Deployment**, kết hợp với cơ chế **Cross-Region Failover** cho tầng frontend.

Tổng quan kiến trúc
-----------------

### Kiến trúc Backend (Single-Region: Singapore)

        ┌─────────────────────────────────────────────────┐
        │                 Người dùng Internet             │
        └───────────────────────┬─────────────────────────┘
                                │
                                ▼
                       ┌────────────────┐
                       │   Route 53     │ ◄── Quản lý DNS
                       │ (Global DNS)   │
                       └────────┬───────┘
                                │
                                ▼
                    ┌──────────────────────┐
                    │  Application Load    │
                    │  Balancer (ALB)      │ ◄── Kết thúc SSL/TLS
                    │  + ACM Certificate   │     (sgutodolist.com)
                    └───────────┬──────────┘
                                │
                ┌───────────────┼──────────────┐
                │               │              │
                ▼               ▼              ▼
             ┌─────────┐   ┌─────────┐   ┌──────────┐
             │  Auth   │   │  User   │   │ Taskflow │ ◄── ECS Fargate
             │ Service │   │ Service │   │ Service  │     Microservices
             └────┬────┘   └────┬────┘   └────┬─────┘     (Public Subnets)
                  │             │             │
                  └─────────────┼─────────────┘
                                │
                     ┌──────────┴──────────┐
                     │     API Gateway     │ ◄── Điểm truy cập trung tâm
                     │     (Port 8080)     │     + CORS + Rate Limiting
                     └──────────┬──────────┘
                                │
                ┌───────────────┼──────────────┐
                │               │              │
                ▼               ▼              ▼
            ┌─────────────┐ ┌─────────┐  ┌──────────┐
            │ Notification│ │  Kafka  │  │ AI Model │
            │   Service   │ │ Cluster │  │ Service  │
            └──────┬──────┘ └────┬────┘  └─────┬────┘
                   │             │             │
                   └─────────────┼─────────────┘
                                 │
                      ┌──────────┴──────────┐
                      │                     │
                      ▼                     ▼
                 ┌─────────┐          ┌───────────┐
                 │   RDS   │          │  Redis    │ ◄── Tầng dữ liệu
                 │  MySQL  │          │ElastiCache│     (Private Subnets)
                 └─────────┘          └───────────┘

### Kiến trúc Frontend (Multi-Region Failover)

        ┌─────────────────────────────────────────────────────────┐
        │                    Người dùng Internet                  │
        └──────────────────────────┬──────────────────────────────┘
                                   │
                                   ▼
                           ┌──────────────┐
                           │   Route 53   │ ◄── DNS: sgutodolist.com
                           └────────┬─────┘
                                    │
                                    ▼
                           ┌────────────────┐
                           │  CloudFront    │ ◄── CDN toàn cầu
                           │  Distribution  │     SSL Certificate
                           └────────┬───────┘     Custom Domain
                                    │
                      ┌─────────────┴─────────────┐
                      │     Origin Group (HA)     │
                      │   ┌─────────────────┐     │
                      │   │ Primary Origin  │     │
                      │   │  S3 Singapore   │ ◄───┼── Vùng chính
                      │   └─────────────────┘     │
                      │            │              │
                      │   ┌────────▼──────────┐   │
                      │   │ Failover Origin   │   │
                      │   │  S3 N. Virginia   │ ◄─┼── Vùng dự phòng
                      │   └───────────────────┘   │
                      └───────────────────────────┘
                                    │
                        ┌───────────┴───────────┐
                        │ S3 Cross-Region       │
                        │ Replication (CRR)     │ ◄── Đồng bộ tự động
                        └───────────────────────┘

Thành phần chính
----------------

### **1. Backend Services (ECS Fargate)**

| Dịch vụ               | Port | Chức năng                                                                 | Phụ thuộc             |
|----------------------|------|---------------------------------------------------------------------------|----------------------|
| API Gateway          | 8080 | - Điều phối routing<br>- Xử lý CORS<br>- Giới hạn tần suất (Rate limiting) | Redis, Tất cả service |
| Auth Service         | 9999 | - Xác thực JWT<br>- OAuth2 (Google)<br>- Quản lý token                     | RDS, Redis, Kafka    |
| User Service         | 8081 | - Quản lý thông tin người dùng<br>- CRUD User                              | RDS, Redis, Kafka    |
| Taskflow Service     | 8082 | - Quản lý công việc<br>- Workflow                                          | RDS, Redis, Kafka    |
| Notification Service | 9998 | - WebSocket realtime<br>- Push notification                                | RDS, Redis, Kafka    |
| AI Model Service     | 9997 | - Ưu tiên task<br>- Gợi ý thông minh                                       | Python Flask         |

### **2. Thành phần hạ tầng**

#### **Networking**

- **VPC**: `10.0.0.0/16` tại Singapore (`ap-southeast-1`)
- **Public Subnets (2 AZs)**: ECS Tasks, ALB, Bastion
- **Private Subnets (2 AZs)**: RDS, Redis (tầng dữ liệu)
- **Chiến lược bảo mật**: Loại bỏ NAT Gateway để tiết kiệm chi phí (~30 USD/tháng)

#### **Lưu trữ dữ liệu**

- **RDS MySQL**: CSDL chính, Free Tier (`db.t3.micro`)
- **ElastiCache Redis**: Cache + Rate limiting (`cache.t3.micro`)
- **Kafka (ECS)**: Event streaming, Service Discovery (`kafka.sgu.local`)

#### **Cân bằng tải & SSL**

- **Application Load Balancer**: Kết thúc HTTPS, routing theo path
- **ACM Certificate**: `sgutodolist.com`
- **Target Groups**: Mỗi service có target group riêng với health check

#### **Service Discovery**

- **AWS Cloud Map**: Namespace `sgu.local`
- **DNS nội bộ**:
  - `auth.sgu.local:9999`
  - `user.sgu.local:8081`
  - `taskflow.sgu.local:8082`
  - `notification.sgu.local:9998`
  - `ai-model.sgu.local:9997`
  - `kafka.sgu.local:9092`

Chi phí & Tối ưu
----------------

### **Các quyết định kiến trúc quan trọng**

| Yếu tố                    | Lựa chọn ban đầu      | Lựa chọn cuối cùng        | Tiết kiệm        |
|---------------------------|-----------------------|---------------------------|------------------|
| **NAT Gateway**           | 2 NAT (HA)            | ❌ Không dùng             | ~60 USD/tháng    |
| **ECS Compute**           | EC2 Instances         | ✅ Fargate Spot (0.5 vCPU)| ~40 USD/tháng    |
| **RDS**                   | Multi-AZ              | ✅ Single-AZ Free Tier    | ~30 USD/tháng    |
| **Redis**                 | Cluster Mode          | ✅ Single Node            | ~20 USD/tháng    |
| **Backend Multi-Region**  | Active-Active         | ✅ Single Region          | ~200 USD/tháng   |
| **Frontend HA**           | Multi-Region Active   | ✅ Failover Only          | ~50 USD/tháng    |

### **Chiến lược Public Subnet cho ECS**

Thay vì sử dụng NAT Gateway tốn kém, toàn bộ ECS Tasks được triển khai trên **Public Subnets** với **Public IP** bật sẵn. Điều này cho phép:

- ✅ Pull Docker image từ ECR qua Internet
- ✅ Kết nối outbound tới các dịch vụ AWS
- ✅ Tiết kiệm 100% chi phí NAT Gateway
- ⚠️ Đánh đổi: cần cấu hình Security Group chặt chẽ

Chiến lược Domain & SSL
-----------------------

### **Domain Production**

- **Frontend**: `https://sgutodolist.com` (CloudFront + ACM us-east-1)
- **Backend API**: `https://sgutodolist.com` (ALB + ACM ap-southeast-1)

### **Chứng chỉ SSL**

- **Chứng chỉ 1** (us-east-1): Dành cho CloudFront (bắt buộc ở Virginia)  
  - Domain: `sgutodolist.com`, `*.sgutodolist.com`
- **Chứng chỉ 2** (ap-southeast-1): Dành cho ALB Singapore  
  - Domain: `sgutodolist.com`

Kiến trúc bảo mật
-----------------

### **Ma trận Security Group**

| SG Name         | Mục đích                  | Inbound Rules                                                         |
|-----------------|---------------------------|------------------------------------------------------------------------|
| `public-alb-sg` | ALB public facing         | 0.0.0.0/0:443, 0.0.0.0/0:80                                          |
| `ecs-app-sg`    | ECS Tasks                 | ALB:8080-8082, 9998-9999; Self: all ports (inter-service)            |
| `private-db-sg` | RDS + Redis + Kafka       | ecs-app-sg:3306, 6379, 9092                                          |
| `bastion-sg`    | Bastion SSH Jump Host     | My IP:22                                                             |

### **Luồng xác thực**


```
User → CloudFront → API Gateway → Auth Service
                                      ↓
                              JWT + Redis Session
                                      ↓
                              Google OAuth2 (Optional)
```

Thông tin nhóm
--------------

- **Dự án**: SGU TodoList – Hệ thống quản lý công việc
- **Kiến trúc**: Microservices Single-Region (Tối ưu chi phí)
- **Region chính**: Asia Pacific (Singapore) – ap-southeast-1
- **Region dự phòng**: US East (N. Virginia) – us-east-1 (chỉ frontend)
- **Mô hình triển khai**: Tối ưu AWS Free Tier
