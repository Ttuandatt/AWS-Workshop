+++
title = "Chuẩn bị Mạng & Bảo mật (VPC, Security Group)"
weight = 1
chapter = false
pre = " <b> 5.3.2.1. </b> "
alwaysopen = true
+++

Giai đoạn này thiết lập **hạ tầng mạng nền tảng** và **ranh giới bảo mật** cho quá trình triển khai hệ thống Microservices.

## Cấu hình VPC và Subnet

### Bước 1: Tạo Hạ tầng VPC

1. Truy cập **VPC Console** → **Create VPC**
2. Chọn tùy chọn **VPC and more**
3. Cấu hình các tham số cho VPC như sau:

| Tham số | Giá trị | Mục đích |
| --- | --- | --- |
| Tự động tạo Name tag | `SGU-Microservices` | Chuẩn hoá quy ước đặt tên |
| IPv4 CIDR block | `10.0.0.0/16` | Dải mạng private tiêu chuẩn |
| Số Availability Zones | 2 (ap-southeast-1a, ap-southeast-1b) | Đảm bảo tính sẵn sàng cao |
| Số public subnet | 2 | Cho các tài nguyên public-facing |
| Số private subnet | 2 | Cô lập tầng dữ liệu |
| NAT Gateway | Không tạo | Tối ưu chi phí (~ tiết kiệm $30/tháng) |
| VPC Endpoint | Không tạo | Tối ưu chi phí |
| DNS options | Enable DNS hostnames + Enable DNS resolution | Phục vụ service discovery |

4. Nhấn **Create VPC**

### **Kết quả kiến trúc mạng**

VPC: 10.0.0.0/16
├── Public Subnet 1: 10.0.0.0/20 (ap-southeast-1a)
├── Public Subnet 2: 10.0.16.0/20 (ap-southeast-1b)
├── Private Subnet 1: 10.0.128.0/20 (ap-southeast-1a)
└── Private Subnet 2: 10.0.144.0/20 (ap-southeast-1b)

yaml
Copy code

---

## Cấu hình Security Groups

Security Group đóng vai trò như **tường lửa ảo**, kiểm soát lưu lượng **inbound / outbound** cho các tài nguyên AWS.

### **Bước 2: Tạo các Security Group**

Truy cập **VPC** → **Security Groups** → **Create security group**.  
Tạo **4 Security Group** với cấu hình sau:

---

### 🔐 Security Group 1: `public-alb-sg` (Application Load Balancer)

| Tham số | Giá trị |
| --- | --- |
| Name | `public-alb-sg` |
| Description | Security group cho ALB của SGUTODOLIST |
| VPC | `SGU-Microservices-VPC` |

**Inbound Rules:**

| Loại | Giao thức | Cổng | Nguồn | Mục đích |
| --- | --- | --- | --- | --- |
| HTTPS | TCP | 443 | 0.0.0.0/0 | Truy cập HTTPS công khai |
| HTTP | TCP | 80 | 0.0.0.0/0 | HTTP (redirect sang HTTPS) |

---

### 🔐 Security Group 2: `ecs-app-sg` (ECS Application Containers)

| Tham số | Giá trị |
| --- | --- |
| Name | `ecs-app-sg` |
| Description | Security group cho container dịch vụ SGUTODOLIST |
| VPC | `SGU-Microservices-VPC` |

#### **Inbound Rules – Giai đoạn 1 (ALB → Services)**

| Loại | Giao thức | Cổng | Nguồn | Mục đích |
| --- | --- | --- | --- | --- |
| Custom TCP | TCP | 8080 | public-alb-sg | ALB → API Gateway |
| Custom TCP | TCP | 8081 | public-alb-sg | ALB → User Service |
| Custom TCP | TCP | 8082 | public-alb-sg | ALB → Taskflow Service |
| Custom TCP | TCP | 9998 | public-alb-sg | ALB → Notification Service |
| Custom TCP | TCP | 9999 | public-alb-sg | ALB → Auth Service |
| Custom TCP | TCP | 9092 | public-alb-sg | Service gọi Kafka |
| Custom TCP | TCP | 9997 | public-alb-sg | ALB → AI Service |

> ⚠️ **Lưu ý:** Nhấn **Create security group** trước khi sang Giai đoạn 2.

#### **Inbound Rules – Giai đoạn 2 (Giao tiếp nội bộ giữa các service)**

1. Chọn `ecs-app-sg`
2. Edit inbound rules → Add rule
3. Cấu hình rule self-reference:
   - Type: **All TCP**
   - Port range: `0-65535`
   - Source: `ecs-app-sg`
   - Mục đích: Cho phép các container giao tiếp nội bộ

---

### 🔐 Security Group 3: `private-db-sg` (Tầng dữ liệu)

| Tham số | Giá trị |
| --- | --- |
| Name | `private-db-sg` |
| Description | Security group cho RDS, Redis, Kafka |
| VPC | `SGU-Microservices-VPC` |

**Inbound Rules:**

| Loại | Giao thức | Cổng | Nguồn | Mục đích |
| --- | --- | --- | --- | --- |
| MySQL/Aurora | TCP | 3306 | ecs-app-sg | Truy cập RDS |
| Custom TCP | TCP | 6379 | ecs-app-sg | Truy cập Redis |
| Custom TCP | TCP | 9092 | ecs-app-sg | Truy cập Kafka |
| MySQL/Aurora | TCP | 3306 | bastion-sg | Quản trị DB từ Bastion |
| Custom TCP | TCP | 6379 | bastion-sg | Quản trị Redis |
| MySQL/Aurora | TCP | 3306 | 14.186.212.182/32 | Truy cập trực tiếp từ IP cố định |

---

### 🔐 Security Group 4: `bastion-sg` (Jump Host / Bastion)

| Tham số | Giá trị |
| --- | --- |
| Name | `bastion-sg` |
| Description | Security group cho Bastion Host |
| VPC | `SGU-Microservices-VPC` |

**Inbound Rules:**

| Loại | Giao thức | Cổng | Nguồn | Mục đích |
| --- | --- | --- | --- | --- |
| SSH | TCP | 22 | My IP | Truy cập SSH an toàn |

> 🔒 **Khuyến nghị bảo mật:** Thay `My IP` bằng IP public thực tế của bạn.

---

## Checklist Xác nhận Hạ tầng Mạng

Trước khi chuyển sang bước tiếp theo, hãy đảm bảo:

- [ ] VPC được tạo với CIDR `10.0.0.0/16`
- [ ] 2 Public subnet trên 2 AZ khác nhau
- [ ] 2 Private subnet trên 2 AZ khác nhau
- [ ] DNS hostnames & DNS resolution được bật
- [ ] Đầy đủ 4 Security Group với rule chính xác
- [ ] `ecs-app-sg` có rule self-reference

---

<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
  <a href="" style="text-decoration: none; font-weight: bold;">
  </a>
  <a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.2-Infrastructure & ALB Setup (RDS, Redis, Cloud Map, ALB Routing)" %}}" style="text-decoration: none; font-weight: bold;">
    BƯỚC 2: Thiết lập Hạ tầng & ALB ➡
  </a>
</div>
