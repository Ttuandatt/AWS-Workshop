+++
title = "Blog 6"
weight =  6
chapter = false
pre = " <b> 3.6. </b>"
+++

# Xây dựng Ứng Dụng Đa Vùng với Các Dịch vụ AWS – Phần 2: Dữ Liệu và Sao Chép

*Tác giả: Joe Chapman và Sebastian Leks - 12/01/2022*

*Chủ đề: [Amazon Aurora](https://aws.amazon.com/blogs/architecture/category/database/amazon-aurora/), [Amazon DocumentDB](https://aws.amazon.com/blogs/architecture/category/database/amazon-document-db/), [Amazon DynamoDB](https://aws.amazon.com/blogs/architecture/category/database/amazon-dynamodb/), [Amazon ElastiCache](https://aws.amazon.com/blogs/architecture/category/database/amazon-elasticache/), [Amazon RDS](https://aws.amazon.com/blogs/architecture/category/database/amazon-rds/), [Amazon Redshift](https://aws.amazon.com/blogs/architecture/category/analytics/amazon-redshift-analytics/), [Amazon Simple Storage Service (S3)](https://aws.amazon.com/blogs/architecture/category/storage/amazon-simple-storage-services-s3/), [Architecture](https://aws.amazon.com/blogs/architecture/category/architecture/), [AWS Backup](https://aws.amazon.com/blogs/architecture/category/storage/aws-backup/), [AWS DataSync](https://aws.amazon.com/blogs/architecture/category/migration/aws-datasync/), [AWS Global Accelerator](https://aws.amazon.com/blogs/architecture/category/networking-content-delivery/aws-global-accelerator/)*

---

Dữ liệu là trung tâm của các ứng dụng có trạng thái (stateful applications). Mô hình nhất quán dữ liệu sẽ khác nhau khi lựa chọn triển khai trong một Region hay nhiều Region. Trong bài viết này, phần 2/3, chúng ta sẽ tiếp tục chọn lọc các dịch vụ AWS, tập trung vào những dịch vụ hướng dữ liệu với các tính năng gốc (native features) giúp đưa dữ liệu đến đúng nơi cần thiết để hỗ trợ chiến lược đa vùng cho ứng dụng của bạn.

Ở **Phần 1** của loạt blog này, chúng ta đã xem xét cách sử dụng các dịch vụ tính toán (compute), mạng (networking), và bảo mật (security) của AWS để xây dựng nền tảng cho một ứng dụng multi-Region. Ở **Phần 3**, chúng ta sẽ tìm hiểu các dịch vụ quản lý và giám sát ứng dụng của AWS, giúp bạn xây dựng, theo dõi và duy trì ứng dụng multi-Region.

### Những lưu ý khi sao chép dữ liệu

Việc sao chép dữ liệu trên mạng AWS diễn ra rất nhanh, nhưng bạn cần nhớ rằng thời gian di chuyển của gói tin sẽ tăng lên theo khoảng cách vật lý mà gói tin cần đi qua. Vì lý do này, khi xây dựng ứng dụng multi-Region, bạn phải cân nhắc giữa tính nhất quán dữ liệu và hiệu năng.

Khi xây dựng một hệ thống phân tán, cần xem xét định lý CAP (Consistency, Availability, Partition Tolerance). Định lý này chỉ ra rằng một ứng dụng chỉ có thể đạt được 2 trong 3 yếu tố, và bạn phải chấp nhận đánh đổi khi đưa ra lựa chọn:

-  Consistency (Nhất quán) – tất cả các client luôn thấy cùng một trạng thái dữ liệu

-  Availability (Sẵn sàng) – tất cả các client luôn có thể đọc và ghi dữ liệu

-  Partition Tolerance (Chịu lỗi phân mảnh mạng) – hệ thống vẫn hoạt động ngay cả khi có sự cố phân mảnh vật lý trong mạng

Trong các kiến trúc single-Region, việc đạt được Consistency và Availability (CA) là điều thường thấy. Ví dụ: khi một ứng dụng chỉ kết nối đến một cơ sở dữ liệu quan hệ trong cùng một Region.

Tuy nhiên, với multi-Region applications, điều này trở nên khó khăn hơn vì độ trễ phát sinh do việc truyền dữ liệu qua khoảng cách địa lý xa. Vì thế, các hệ thống phân tán ở mức toàn cầu thường chấp nhận hy sinh tính nhất quán tuyệt đối, thay vào đó ưu tiên Availability và Partition Tolerance (AP). Mô hình này thường được gọi là eventual consistency (nhất quán cuối cùng).


{{< figurecaption src="/images/Img1-Blog6.png" caption="" >}}