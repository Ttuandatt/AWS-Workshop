+++
title = "Điều kiện tiên quyết"
weight = 2
chapter = false
pre = " <b> 5.2.  </b> "
+++

### Các điều kiện tiên quyết cho triển khai Backend

Trước khi bắt đầu các bước triển khai được trình bày trong Mục lục, cần hoàn tất các công cụ nền tảng, cấu hình hệ thống và thiết lập dịch vụ AWS dưới đây nhằm đảm bảo quá trình triển khai diễn ra **ổn định, an toàn và tối ưu chi phí** trong khu vực **ap-southeast-1 (Singapore)**.

| **Bước** | **Yêu cầu** | **Mục đích** |
| --- | --- | --- |
| 1 | **Tài khoản AWS & AWS CLI** | Truy cập và quản lý hạ tầng thông qua giao diện dòng lệnh |
| 2 | **Tên miền & SSL** | Bảo mật lưu lượng truy cập và phân phối nội dung toàn cầu |
| 3 | **Mã nguồn & Artifacts** | Xây dựng các image microservice có thể triển khai |
| 4 | **Nền tảng mạng AWS** | Thiết lập kiến trúc VPC cô lập, sẵn sàng cho High Availability |

* * * * *

### I. Công cụ & Thông tin xác thực

1. **Thiết lập tài khoản AWS:**  
   Cần có một tài khoản AWS hợp lệ và phải cấu hình **AWS Budgets (cảnh báo chi phí)** để theo dõi mức sử dụng Free Tier và tránh phát sinh chi phí ngoài ý muốn.

2. **AWS CLI:**  
   AWS Command Line Interface cần được cài đặt và cấu hình với **IAM User credentials** có quyền truy cập thích hợp tới các dịch vụ: EC2, RDS, S3, CloudFront, Route 53, IAM và CloudWatch.

3. **Docker:**  
   Docker Engine cần được cài đặt trên máy local để build các Docker image cho các microservice Spring Boot.

4. **Java / Maven:**  
   Môi trường local phải được thiết lập đầy đủ Java và Maven để biên dịch và đóng gói mã nguồn ứng dụng Spring Boot.

---

### II. Chuẩn bị Tên miền, SSL và DNS

1. **Tên miền đã đăng ký:**  
   Phải sở hữu một tên miền hợp lệ (có thể đăng ký qua Route 53 hoặc nhà cung cấp bên ngoài).

2. **Hosted Zone trên Route 53:**  
   Tên miền cần được quản lý trong **Amazon Route 53 Hosted Zone**.

3. **Chứng chỉ SSL (ACM):**  
   Chứng chỉ **AWS Certificate Manager (ACM)** phải được tạo cho tên miền (ví dụ: `*.taskmanager.com`) tại **hai khu vực**:

   - **us-east-1 (N. Virginia):** Bắt buộc cho CloudFront (dịch vụ toàn cầu).
   - **ap-southeast-1 (Singapore):** Dùng cho Application Load Balancer (ALB).

---

### III. Nền tảng mạng (VPC & Subnet)

Hệ thống VPC phải được tạo tại **ap-southeast-1**, đảm bảo **High Availability (HA)** thông qua tối thiểu hai Availability Zones (AZs), với cấu trúc như sau:

1. **VPC:**  
   Một VPC chính (ví dụ: CIDR `10.0.0.0/16`).

2. **Public Subnets (Multi-AZ):**  
   Dùng để triển khai **Application Load Balancer (ALB)** và **Internet Gateway (IGW)**.

3. **Private Subnets (Multi-AZ):**  
   Dùng để triển khai các tài nguyên backend như **EC2/ECS**, **RDS**, và **ElastiCache**.

4. **NAT Gateway:**  
   Được triển khai trong ít nhất một Public Subnet để cho phép các tài nguyên trong Private Subnet (EC2/ECS) truy cập Internet một cách an toàn (cập nhật hệ thống, kết nối đến S3/ECR, v.v.).

5. **Security Groups (SG):**  
   Các Security Group cần được thiết kế theo nguyên tắc **Least Privilege**:

   - **ALB SG:** Cho phép truy cập từ Internet qua **HTTP (80)** và **HTTPS (443)**.
   - **EC2/ECS SG:** Chỉ cho phép inbound traffic từ **ALB SG** trên cổng ứng dụng (ví dụ: 8080).
   - **RDS / ElastiCache SG:** Chỉ cho phép inbound từ **EC2/ECS SG** trên các cổng dịch vụ tương ứng (MySQL 3306, Redis 6379).

---

### IV. Chuẩn bị dữ liệu & Lưu trữ

1. **S3 Primary Bucket (ap-southeast-1):**  
   Tạo một S3 bucket tại `ap-southeast-1` để lưu trữ static assets và các artifacts triển khai ứng dụng (JAR hoặc Docker image).

2. **S3 DR Bucket (us-east-1):**  
   Tạo một S3 bucket thứ hai tại `us-east-1` dùng làm **Disaster Recovery (DR)**.

3. **Cross-Region Replication (CRR):**  
   Cấu hình **S3 Cross-Region Replication** để tự động sao chép dữ liệu từ bucket chính tại `ap-southeast-1` sang bucket DR tại `us-east-1`.

---

### V. Chuẩn bị Compute & Dịch vụ

1. **Docker Images:**  
   Các microservice Spring Boot phải được build thành Docker image đạt tiêu chuẩn production.

2. **ECR Repository (Tùy chọn):**  
   Nếu sử dụng ECS, cần tạo repository trong **Amazon ECR (Elastic Container Registry)** để lưu trữ Docker image.

3. **IAM Roles:**  
   Cần tạo các **IAM Role** cần thiết cho:

   - **EC2 / ECS Tasks:** Cấp quyền truy cập tới S3, CloudWatch và cơ sở dữ liệu RDS.
   - **ALB:** Cho phép Load Balancer thực hiện đầy đủ chức năng phân phối lưu lượng.
