+++
title = "Triển khai Backend"
weight = 2
chapter = false
pre = " <b> 5.3.2. </b> "
alwaysopen = true
+++

### **Mô hình kiến trúc**

Mô hình này minh họa các dịch vụ cốt lõi được sử dụng nhằm đảm bảo **tính sẵn sàng cao**, **độ trễ thấp** và **bảo mật** cho hệ thống:

1. **Tên miền & SSL:** Route 53 (DNS) được dùng để quản lý tên miền, trong khi ACM (SSL Certificate) đảm bảo mã hóa và bảo mật lưu lượng truy cập.

2. **CDN:** CloudFront đóng vai trò là mạng phân phối nội dung toàn cầu (Global Edge Network).

3. **Lưu trữ chính:** Amazon S3 tại Singapore (`ap-southeast-1`) lưu trữ các tài nguyên tĩnh chính của hệ thống.

4. **Lưu trữ dự phòng (Failover):** Amazon S3 tại N. Virginia (`us-east-1`) hoạt động như vùng lưu trữ dự phòng.

5. **Sao chép dữ liệu:** Cơ chế replication tự động sao chép dữ liệu từ Singapore sang Virginia nhằm đảm bảo tính dự phòng và khả năng khôi phục.

6. **Bảo mật:** OAC (Origin Access Control) được sử dụng để giữ S3 bucket ở trạng thái private và đảm bảo lưu lượng chỉ được truy cập thông qua CloudFront.

### **Mục lục nội dung**

1. [Chuẩn bị mạng & bảo mật (VPC, Security Group)]({{% relref "5.3.2.1-Network & Security Preparation (VPC, SG)" %}})

2. [Thiết lập hạ tầng & ALB (RDS, Redis, Cloud Map, điều hướng ALB)]({{% relref "5.3.2.2-Infrastructure & ALB Setup (RDS, Redis, Cloud Map, ALB Routing)" %}})

3. [Cập nhật mã nguồn & build image (Tạo Docker Image mới)]({{% relref "5.3.2.3-Code Update & Image Build (Create new Docker Image)" %}})

4. [Tạo Task Definition (Cấu hình, sửa biến môi trường)]({{% relref "5.3.2.4-Task Definitions Creation (Configure settings, FIX environment variables)" %}})

5. [Triển khai Service]({{% relref "5.3.2.5-Services Deployment" %}})

6. [Hoàn tất & xác minh (Route 53, Google Console, kiểm thử cuối cùng)]({{% relref "5.3.2.6-Completion & Verification (Route 53, Google Console, Final Test)" %}})
