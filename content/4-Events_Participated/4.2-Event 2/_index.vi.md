+++
title = "Sự kiện 2"
weight = 2
chapter = false
pre = " <b> 4.2.  </b> "
+++

# AWS Cloud Mastery Series #3 – Security Pillar Workshop

Buổi workshop tập trung giúp người tham gia hiểu cách **thiết kế và vận hành hệ thống AWS an toàn** dựa trên **Security Pillar của AWS Well-Architected Framework**, kết hợp giữa lý thuyết, demo thực tế và các tình huống trong môi trường triển khai thật.

---

## Những Gì Tôi Học Được

### Nền Tảng Bảo Mật (Security Foundation)

Phiên đầu tiên giới thiệu tổng quan về **Security Pillar** và vai trò của nó trong kiến trúc cloud. Các nguyên tắc bảo mật cốt lõi như **Least Privilege, Zero Trust và Defense in Depth** được giải thích theo góc nhìn thực tiễn.  
Diễn giả cũng làm rõ **AWS Shared Responsibility Model**, chỉ ra những hiểu lầm phổ biến và các rủi ro bảo mật thường gặp trong môi trường cloud tại Việt Nam.

---

### Quản Lý Danh Tính & Truy Cập (Identity & Access Management – IAM)

Phần này trình bày cách thiết kế IAM hiện đại và các best practices, bao gồm:
- Sử dụng **IAM Role thay cho access key dài hạn**
- Quản lý quyền với **IAM User, Role và Policy**
- Triển khai SSO và phân quyền tập trung bằng **IAM Identity Center**
- Áp dụng **Service Control Policy (SCP)** và permission boundary trong mô hình multi-account
- Tăng cường bảo mật với **MFA, luân chuyển credential và IAM Access Analyzer**

Một demo ngắn minh họa cách **kiểm tra IAM policy và mô phỏng quyền truy cập**, cho thấy chỉ một cấu hình sai nhỏ cũng có thể dẫn đến cấp quyền quá mức.

---

### Phát Hiện & Giám Sát (Detection & Monitoring)

Workshop tiếp tục với nội dung về khả năng giám sát và phát hiện sự cố bảo mật trên AWS:
- Sử dụng **CloudTrail, GuardDuty và Security Hub** để phát hiện liên tục
- Triển khai logging đa tầng (VPC Flow Logs, ALB logs, S3 logs)
- Tự động cảnh báo và phản hồi bằng **EventBridge**
- Áp dụng khái niệm **Detection-as-Code** để đảm bảo tính nhất quán và tái sử dụng

---

### Bảo Vệ Hạ Tầng (Infrastructure Protection)

Phần này tập trung vào bảo mật tầng mạng và compute:
- Thiết kế VPC với **phân tách rõ ràng public/private**
- Hiểu sự khác nhau giữa **Security Group và Network ACL**
- Bảo vệ hệ thống với **WAF, Shield và Network Firewall**
- Các nguyên tắc bảo mật cơ bản cho **EC2 và container (ECS/EKS)**

Những ví dụ thực tế giúp liên kết rõ ràng giữa quyết định kiến trúc và kết quả bảo mật đạt được.

---

### Bảo Vệ Dữ Liệu (Data Protection)

Nội dung bảo mật dữ liệu được trình bày thông qua các chiến lược mã hóa và kiểm soát truy cập:
- Quản lý khóa mã hóa bằng **AWS KMS** (policy, grant, rotation)
- Mã hóa dữ liệu **at-rest và in-transit** cho các dịch vụ như S3, EBS, RDS, DynamoDB
- Lưu trữ và xoay vòng secrets với **Secrets Manager** và **Parameter Store**
- Áp dụng **phân loại dữ liệu và guardrail** nhằm giảm thiểu rủi ro rò rỉ

---

### Ứng Phó Sự Cố (Incident Response)

Phiên kỹ thuật cuối cùng tập trung vào **Incident Response**:
- Vòng đời IR theo khuyến nghị của AWS
- Playbook cho các tình huống thực tế:
  - Lộ thông tin IAM credential
  - S3 bucket bị public ngoài ý muốn
  - Phát hiện malware trên EC2
- Kỹ thuật cô lập tài nguyên, snapshot và thu thập bằng chứng
- Tự động hóa phản hồi với **Lambda** và **Step Functions**

Phần này nhấn mạnh tầm quan trọng của việc **chuẩn bị sẵn kịch bản ứng phó trước khi sự cố xảy ra**.

---

## Tổng Kết

- Tôi hiểu rõ cách các **trụ cột bảo mật trong AWS** phối hợp với nhau trong một hệ thống hoàn chỉnh.
- Bảo mật không chỉ là công cụ, mà là **thiết kế đúng, tự động hóa và giám sát liên tục**.
- Thiết kế IAM chuẩn là yếu tố then chốt và cũng là điểm yếu phổ biến nhất trong môi trường cloud.
- Incident Response cần được chuẩn bị và tự động hóa, không nên xử lý thủ công khi sự cố đã xảy ra.

> Nhìn chung, workshop đã cung cấp nhiều kiến thức thực tiễn về bảo mật AWS, giúp tôi tự tin hơn trong việc thiết kế và vận hành các hệ thống cloud an toàn.
