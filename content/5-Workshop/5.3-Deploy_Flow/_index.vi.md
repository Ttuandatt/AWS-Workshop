+++
title = "Quy trình triển khai"
weight = 3
chapter = false
pre = " <b> 5.3.  </b> "
+++

### **Mục lục**

Phần này cung cấp **hướng dẫn từng bước** để triển khai cả **Frontend** và **Backend** lên hạ tầng điện toán đám mây.  
Nội dung bao quát **toàn bộ quy trình triển khai**, từ khâu chuẩn bị build artifacts cho đến khi ứng dụng được phát hành ở môi trường production trên các dịch vụ AWS.

**5.3.1.** [Triển khai Frontend]({{< relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/_index.vi.md" >}})  
&nbsp;&nbsp;Hướng dẫn build, cấu hình và triển khai frontend lên **Amazon S3 + CloudFront**, bao gồm cơ chế **failover đa vùng (cross-region)**.

**5.3.2.** [Triển khai Backend]({{< relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/_index.vi.md" >}})  
&nbsp;&nbsp;Hướng dẫn triển khai các backend microservices lên **AWS ECS/Fargate**, bao gồm cấu hình mạng, cân bằng tải và tích hợp giữa các dịch vụ.

> Sau khi hoàn thành phần này, ứng dụng sẽ được triển khai đầy đủ trên AWS với kiến trúc **bảo mật**, **có khả năng mở rộng** và **tối ưu chi phí**.