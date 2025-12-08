+++
title = "Workshop"
weight = 5
chapter = false
pre = "<b>5. </b>"
+++

# **Nền tảng SaaS Quản Lý Công Việc Tối Ưu Chi Phí**

### **Tổng quan**
**SGU TodoList** là một ứng dụng quản lý công việc có tính ổn định cao và khả năng chịu lỗi tốt, được xây dựng theo **kiến trúc Microservices** nhằm đảm bảo khả năng mở rộng và tách biệt các thành phần.  
Mục tiêu chính của workshop này là hướng dẫn người học **triển khai và quản lý một hệ thống ứng dụng phức tạp trên nền tảng AWS Cloud**, với trọng tâm vào **tối ưu chi phí** và **vận hành theo mô hình serverless**.

Thông qua việc tận dụng các dịch vụ cốt lõi của AWS như **ECS Fargate**, **S3**, **ALB** và **VPC**, bạn sẽ học cách thiết lập một hạ tầng có **hiệu năng cao**, **tính sẵn sàng cao**, đồng thời **giảm thiểu chi phí vận hành** so với việc sử dụng EC2 truyền thống.

### **Công nghệ và khái niệm chính**
Workshop này tập trung vào triển khai thực tế với các công nghệ và khái niệm:
* **Kiến trúc:** Microservices, Event-Driven (Kafka).
* **Tài nguyên tính toán:** AWS ECS Fargate (Container Serverless).
* **Backend:** Spring Boot (Java 21).
* **Frontend:** ReactJS.
* **Hệ thống dữ liệu:** AWS RDS (MySQL), ElastiCache (Redis).
* **Mạng & khám phá dịch vụ:** AWS ALB, AWS Cloud Map (Service Discovery).
* **Triển khai:** AWS CLI và các script tự động bằng PowerShell.

### **Mục lục nội dung**
**1.** [Tổng quan Workshop]({{< relref "5-Workshop/5.1-Workshop_Overview/_index.md" >}}) – *Mục tiêu và kết quả học tập đạt được.*

**2.** [Điều kiện tiên quyết]({{< relref "5-Workshop/5.2-Prerequisite/_index.md" >}}) – *Các công cụ và quyền AWS cần thiết để triển khai.*

**3.** [Quy trình triển khai]({{< relref "5-Workshop/5.3-Deploy_Flow/_index.md" >}}) – *Hướng dẫn từng bước triển khai Backend và Frontend.*

**4.** [Dọn dẹp tài nguyên]({{< relref "5-Workshop/5.4-Clean_Up/_index.md" >}}) – *Hướng dẫn xóa toàn bộ tài nguyên nhằm tránh phát sinh chi phí.*

**5.** [Bí thuật]({{< relref "5-Workshop/5.5-Mystic Skills/_index.md" >}}) – *Bí thuật.*
