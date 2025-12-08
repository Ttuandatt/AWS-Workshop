+++
title = "Dọn Dẹp"
weight = 4
chapter = false
pre = " <b> 5.4.  </b> "
+++

Hướng Dẫn Dọn Dẹp
-----------------------------------------------------------

Hướng dẫn này cung cấp các bước chi tiết để **xóa hoàn toàn tất cả tài nguyên AWS** đã triển khai trên các vùng **ap-southeast-1 (Singapore)** và **us-east-1 (N. Virginia)**. Thứ tự thực hiện được tối ưu để tránh tài nguyên thừa và các chi phí phát sinh không mong muốn.

### **I. Tổng Quan Dịch Vụ AWS Đang Hoạt Động**

Trước khi thực hiện, hãy kiểm tra sự tồn tại của các dịch vụ sau trong tài khoản:

| Danh Mục Dịch Vụ | Tên Dịch Vụ/Tài Nguyên | Vùng | Mục Đích |
| --- | --- | --- | --- |
| **Compute/Scaling** | ECS Cluster/ASG, EC2 Instances | ap-southeast-1 | Lưu trữ ứng dụng |
| **Cơ Sở Dữ Liệu/Cache** | RDS DB Instance (Multi-AZ), ElastiCache Redis Cluster | ap-southeast-1 | Lưu trữ dữ liệu & Quản lý phiên |
| **Mạng** | Application Load Balancer (ALB), Target Groups, NAT Gateway, VPC | ap-southeast-1 | Phân phối lưu lượng & Truy cập Internet |
| **Lưu Trữ/Tài Nguyên** | S3 Primary Bucket, S3 DR Bucket, ECR Repositories | ap-southeast-1, us-east-1 | Lưu trữ file & hosting image |
| **Toàn Cầu/DNS/Bảo Mật** | CloudFront Distribution, ACM Certificates, Route 53 Hosted Zone | Global (Edge), us-east-1, ap-southeast-1 | CDN & bảo mật SSL |

* * * * *

### **II. Quy Trình Dọn Dẹp (Từng Bước)**

#### **1\. Dọn Dẹp Lớp Ứng Dụng (ap-southeast-1)**

Bắt đầu với việc xóa các tài nguyên chạy ứng dụng và kết nối đến cơ sở dữ liệu.

1.  **Dừng ECS/EC2 Compute:**  

    -   **Nếu sử dụng ECS:**  
        -   Vào **Amazon ECS** → **Clusters**.  
        -   Chọn Cluster → Tab **Services**.  
        -   Chọn tất cả Services → Click **Update**.  
        -   Đặt **Desired tasks** = **0** → **Next** → **Update Service**.  
        -   Chờ tasks dừng hoàn toàn. Sau đó chọn lại Services → **Delete** → Xác nhận với `delete me`.

2.  **Xóa Thành Phần Load Balancer (ALB):**  

    -   Vào **EC2** → **Target Groups**. Chọn Target Groups → **Actions** → **Delete**.  

    -   Vào **EC2** → **Load Balancers**. Chọn ALB → **Actions** → **Delete**.

#### **2\. Xóa Dịch Vụ Cơ Sở Dữ Liệu và Cache**

1.  **Xóa ElastiCache (Redis):**  

    -   Vào **ElastiCache** → **Redis Clusters**. Chọn Cluster → **Actions** → **Delete**.

2.  **Xóa RDS Database (Bước quan trọng về chi phí):**  

    -   Vào **RDS** → **Databases**. Chọn Instance.  

    -   Nếu **Deletion protection** đang *Enabled*, click **Modify** → Bỏ chọn *Deletion protection* → **Continue** → **Apply immediately**.  

    -   Chọn Instance → **Actions** → **Delete**.  

    -   **Bỏ chọn** **Create final snapshot** → Nhập tên DB để xác nhận → Click **Delete**.

#### **3\. Dọn Dẹp Dịch Vụ Toàn Cầu và Certificates**

Bao gồm các tài nguyên trải dài nhiều vùng.

1.  **Xóa CloudFront Distribution:**  

    -   Vào **CloudFront**. Chọn Distribution → Click **Disable**. (Chờ 10-15 phút để trạng thái thay đổi).  

    -   Khi Disabled, chọn Distribution → Click **Delete**.

2.  **Xóa ACM Certificates (cần thực hiện ở cả hai vùng):**  

    -   **us-east-1 (N. Virginia):** Chuyển vùng → Vào **ACM**. Chọn Certificate → Click **Delete**.  

    -   **ap-southeast-1 (Singapore):** Chuyển vùng → Vào **ACM**. Chọn Certificate → Click **Delete**.

#### **4\. Dọn Dẹp Lưu Trữ và Artifacts**

1.  **Xóa ECR Repositories:**  

    -   Vào **Elastic Container Registry (ECR)**. Chọn Repositories → Click **Delete** → Nhập `delete` để xác nhận.

2.  **Dọn dẹp và Xóa S3 Buckets:**  

    -   **Loại bỏ CRR:** Vào **S3** → Chọn Primary Bucket (ap-southeast-1) → Tab **Management** → **Replication Rules** → **Delete** rule.  

    -   **Xóa nội dung:** Với cả hai S3 Buckets, vào Tab **Objects** → Chọn tất cả → Click **Delete** → Nhập `permanently delete` để xác nhận.  

    -   **Xóa Bucket:** Sau khi empty, vào Tab **Properties** → Click **Delete** → Nhập tên Bucket để xác nhận.

#### **5\. Xóa Mạng và DNS**

1.  **Xóa NAT Gateway (Bước quan trọng về chi phí):**  

    -   Vào **VPC** → **NAT Gateways**. Chọn Gateway → **Actions** → **Delete NAT Gateway**.

2.  **Xóa Internet Gateway (IGW):**  

    -   Vào **VPC** → **Internet Gateways**. Chọn IGW → **Actions** → **Detach from VPC**.  

    -   Chọn IGW → **Actions** → **Delete Internet Gateway**.

3.  **Xóa các thành phần VPC:**  

    -   Xóa **Security Groups** → Xóa **Subnets** → Xóa **VPC**.

4.  **Xóa Route 53 Hosted Zone:**  

    -   Vào **Route 53** → **Hosted Zones**. Xóa tất cả các custom records → Click **Delete Hosted Zone**.

* * * * *

### **III. Kiểm Tra Cuối Cùng**

-   **AWS Billing/Cost Explorer:** Kiểm tra ngay để đảm bảo chi phí ước tính giờ đã giảm xuống **$0.00**.  

-   **Dashboard:** Xác nhận trên EC2, ECS, RDS ở **ap-southeast-1** không còn tài nguyên đang chạy.
