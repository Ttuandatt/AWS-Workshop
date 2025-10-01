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

{{< figurecaption src="/images/Img1-Blog6.png" caption="" >}}

Trong các kiến trúc single-Region, việc đạt được Consistency và Availability (CA) là điều thường thấy. Ví dụ: khi một ứng dụng chỉ kết nối đến một cơ sở dữ liệu quan hệ trong cùng một Region.

Tuy nhiên, với multi-Region applications, điều này trở nên khó khăn hơn vì độ trễ phát sinh do việc truyền dữ liệu qua khoảng cách địa lý xa. Vì thế, các hệ thống phân tán ở mức toàn cầu thường chấp nhận hy sinh tính nhất quán tuyệt đối, thay vào đó ưu tiên Availability và Partition Tolerance (AP). Mô hình này thường được gọi là eventual consistency (nhất quán cuối cùng).


### Sao chép đối tượng và tệp tin
Quy mô, độ bền và tính sẵn sàng của [Amazon Simple Storage Service (Amazon S3)](http://aws.amazon.com/s3) khiến nó trở thành một điểm đến tuyệt vời để lưu trữ nhiều loại tài nguyên cho ứng dụng của bạn. Để cho phép ứng dụng truy cập nhanh dữ liệu này bất kể được triển khai ở Region nào, bạn có thể thiết lập Amazon S3 để [sao chép dữ liệu giữa các Region AWS](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html) với cơ chế sao chép một chiều hoặc [hai chiều](https://wellarchitectedlabs.com/reliability/200_labs/200_bidirectional_replication_for_s3/) liên tục. Nếu ứng dụng của bạn không cần tất cả các đối tượng trong một bucket để hoạt động tối ưu, một tập con đối tượng có thể được sao chép bằng các [quy tắc replication](https://docs.aws.amazon.com/AmazonS3/latest/userguide/disable-replication.html) của Amazon S3. Amazon S3 trong một Region cung cấp tính nhất quán mạnh khi đọc ngay sau khi ghi; tuy nhiên, các đối tượng đã được sao chép sẽ chỉ nhất quán sau một khoảng thời gian ở các Region đích. Nếu việc duy trì độ trễ sao chép thấp là quan trọng, [S3 Replication Time Control](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication-walkthrough-5.html) sẽ sao chép 99,99% đối tượng trong vòng 15 phút, và hầu hết chỉ mất vài giây. Quan sát là yếu tố then chốt ở mọi lớp của một ứng dụng, và bạn có thể giám sát trạng thái sao chép của các đối tượng bằng Amazon [S3 events và metrics](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication-metrics.html).

Mỗi bucket S3 được cấp endpoint riêng theo Region, điều này có nghĩa là endpoint bao gồm Region của bucket trong URL, ví dụ:

```
https://<MY-BUCKET>.s3.<AWS-REGION>.amazonaws.com/
```

Để đơn giản hóa việc kết nối đến và quản lý nhiều endpoint bucket, Amazon [S3 Multi-Region Access Points](https://aws.amazon.com/blogs/aws/s3-multi-region-access-points-accelerate-performance-availability/) tạo ra một endpoint toàn cầu duy nhất bao phủ nhiều bucket S3 ở các Region khác nhau. Khi ứng dụng kết nối đến endpoint này, các yêu cầu sẽ được định tuyến qua mạng AWS bằng [AWS Global Accelerator](https://aws.amazon.com/global-accelerator/) đến bucket có độ trễ thấp nhất. Việc định tuyến dự phòng cũng được xử lý tự động nếu khả năng sẵn sàng của một bucket thay đổi. Bạn sẽ thấy rằng các endpoint này không phụ thuộc Region trong URL:

```
https://<RANDOM-STRING>.mrap.s3-global.amazonaws.com/
```
Nếu một số phần của ứng dụng bạn chưa hỗ trợ đọc và ghi vào object store trên nền web như Amazon S3, bạn có thể có cùng độ bền và tính sẵn sàng trên một hệ thống tệp chia sẻ với các lớp lưu trữ theo Region của [Amazon Elastic File System (Amazon EFS)](http://aws.amazon.com/efs). Để đảm bảo các triển khai ứng dụng đa Region có thể truy cập dữ liệu này tại chỗ, bạn có thể thiết lập [replication dựa trên block của Amazon EFS](https://docs.aws.amazon.com/efs/latest/ug/efs-replication.html) đến một volume EFS chỉ đọc ở Region khác.

Đối với các tệp được lưu bên ngoài Amazon S3 và Amazon EFS, [AWS DataSync](https://aws.amazon.com/datasync/) đơn giản hóa, tự động hóa và tăng tốc việc [di chuyển dữ liệu giữa các Region và account](https://aws.amazon.com/blogs/storage/transferring-file-data-across-aws-regions-and-accounts-using-aws-datasync/). Nó hỗ trợ cả di chuyển đồng nhất và khác loại giữa [Amazon FSx](https://aws.amazon.com/fsx/), [AWS Snowcone](https://aws.amazon.com/snowcone/), Amazon EFS, và Amazon S3. Nó thậm chí có thể được dùng để đồng bộ các tệp on-premises được lưu trữ trên NFS, SMB, HDFS và object storage tự quản lý lên AWS cho các kiến trúc hybrid.

Việc sao chép tệp và đối tượng được kỳ vọng là sẽ nhất quán sau một khoảng thời gian. Tốc độ mà một tập dữ liệu có thể truyền tải phụ thuộc vào dung lượng dữ liệu, băng thông I/O, băng thông mạng, điều kiện mạng và khoảng cách vật lý.

### Sao chép bản sao lưu

Bên cạnh việc sao chép dữ liệu (replication), bạn cũng nên sao lưu dữ liệu của mình. Trong trường hợp dữ liệu bị hỏng hoặc vô tình bị xóa, một bản sao lưu sẽ cho phép bạn khôi phục về trạng thái tốt cuối cùng đã biết. Ngoài ra, không phải tất cả dữ liệu đều cần được sao chép theo thời gian thực. Ví dụ, nếu bạn đang thiết kế cho kịch bản khôi phục thảm họa và ứng dụng của bạn có [RTO (Recovery Time Objective – Mục tiêu thời gian khôi phục) và RPO (Recovery Point Objective – Mục tiêu điểm khôi phục)](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/disaster-recovery-dr-objectives.html) dài hơn, thì việc sao chép bản sao lưu theo lịch trình có thể đáp ứng yêu cầu của bạn trong khi lại tiết kiệm chi phí hơn.
Đối với những trường hợp này, [AWS Backup](https://aws.amazon.com/backup/) có thể tự động hóa việc sao lưu dữ liệu của bạn để đáp ứng yêu cầu kinh doanh, và các bản sao lưu này có thể được cấu hình để tự động [sao chép đến một hoặc nhiều Region hoặc account AWS](https://docs.aws.amazon.com/aws-backup/latest/devguide/cross-region-backup.html). Thời gian sao chép phụ thuộc vào kích thước bản sao lưu của bạn và khoảng cách cần truyền tải. Bạn nên chạy thử nghiệm trước để xác định xem thời gian sao chép xuyên Region này có ảnh hưởng đến RTO và RPO đã định nghĩa của bạn hay không. Ngày càng có nhiều dịch vụ được hỗ trợ, và điều này đặc biệt hữu ích cho các dịch vụ không cung cấp tính năng sao chép theo thời gian thực sang Region khác như [Amazon Elastic Compute Cloud (Amazon EC2)](http://aws.amazon.com/ec2) hoặc [Amazon Neptune](https://aws.amazon.com/neptune/).

Hình 1 cho thấy cách các dịch vụ dữ liệu được đề cập có thể được kết hợp cho từng tài nguyên trong một kiến trúc. Chúng tôi sử dụng Amazon S3 bi-directional replication để giữ cho các đối tượng trong bucket được đồng bộ trong khi vẫn cho phép ghi từ bất kỳ Region nào. Các máy chủ ứng dụng dựa trên Amazon EC2 của chúng tôi có một mount Amazon EFS bao gồm các bản nhị phân ứng dụng mới nhất và các tệp hỗ trợ, và chúng tôi dựa vào replication của Amazon EFS để giữ dữ liệu này được đồng bộ trên các triển khai ở nhiều Region. Cuối cùng, chúng tôi sử dụng AWS Backup để tạo snapshot cho các instance EC2 của mình và lưu chúng dưới dạng AMI images trong Region thứ cấp.

{{< figurecaption src="/images/Img2-Blog6.png" caption="Hình 1. Các dịch vụ sao lưu lưu trữ" >}}


### Mở rộng cơ sở dữ liệu phi quan hệ trên nhiều Region

[Amazon DynamoDB](https://aws.amazon.com/dynamodb/) [global tables](https://aws.amazon.com/dynamodb/global-tables/) cung cấp khả năng đa Region và đa điểm ghi, giúp bạn xây dựng các ứng dụng toàn cầu ở quy mô lớn. DynamoDB global table là dịch vụ cơ sở dữ liệu do AWS quản lý duy nhất cho phép nhiều điểm ghi chủ động với khả năng phát hiện xung đột trong kiến trúc đa Region (active-active và multi-Region). Điều này cho phép ứng dụng đọc và ghi dữ liệu tại Region gần nhất, đồng thời các thay đổi sẽ tự động được sao chép sang các Region khác.
Việc đọc toàn cầu và khôi phục nhanh cho [Amazon DocumentDB (tương thích với MongoDB)](https://aws.amazon.com/documentdb/) có thể đạt được thông qua các [global cluster](https://docs.aws.amazon.com/documentdb/latest/developerguide/global-clusters.html). Các cluster này có một Region chính xử lý thao tác ghi. Hạ tầng sao chép dựa trên lưu trữ chuyên biệt giúp thực hiện đọc toàn cầu với độ trễ thường dưới một giây.

Giữ bộ nhớ đệm trong RAM (in-memory cache) được đồng bộ dữ liệu trên nhiều Region là yếu tố quan trọng để duy trì hiệu năng ứng dụng. [Amazon ElastiCache](https://aws.amazon.com/elasticache/redis/) (Redis OSS) cung cấp [global datastore](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Redis-Global-Datastore.html) để tạo một bản sao chéo Region được quản lý toàn diện, nhanh chóng, đáng tin cậy và an toàn cho Redis OSS caches và databases. Với global datastore, các thao tác ghi ở một Region có thể được đọc từ tối đa hai cluster bản sao ở Region khác – loại bỏ nhu cầu phải ghi vào nhiều cache để giữ chúng luôn “ấm”.

### Mở rộng cơ sở dữ liệu quan hệ trên nhiều Region

Đối với các ứng dụng cần mô hình dữ liệu quan hệ, [Amazon Aurora](https://aws.amazon.com/rds/aurora/) [global database](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html) hỗ trợ mở rộng khả năng đọc dữ liệu trên nhiều Region trong cả hai phiên bản tương thích Aurora PostgreSQL và MySQL. Hạ tầng sao chép chuyên biệt sử dụng cơ chế sao chép dựa trên lưu trữ vật lý, giúp duy trì độ trễ sao chép ổn định và thấp, vượt trội hơn so với các cơ chế sao chép logic tích hợp sẵn của các hệ quản trị cơ sở dữ liệu, như minh họa ở Hình 2.

{{< figurecaption src="/images/Img3-Blog6.png" caption="Hình 2. SysBench OLTP (chỉ ghi) tăng dần mỗi 600 giây trên R4.16xlarge">}}

Với Aurora global database, một cluster ở một Region được chỉ định là writer, trong khi các Region phụ được dành cho việc đọc. Mặc dù chỉ có một instance duy nhất xử lý các thao tác ghi, Aurora MySQL hỗ trợ [write forwarding](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database-write-forwarding.html), một tính năng sẽ chuyển các truy vấn ghi từ endpoint của Region phụ sang Region chính, giúp đơn giản hóa logic trong mã ứng dụng.
Bạn có thể kiểm tra Regional failover bằng cách sử dụng [managed planned failover](https://aws.amazon.com/blogs/database/managed-planned-failovers-with-amazon-aurora-global-database/) của Aurora global database. Hành động này sẽ chuyển cluster ghi chủ động sang một Region phụ trong khi vẫn giữ nguyên kiến trúc sao chép. Cần lưu ý rằng hành động này phụ thuộc vào việc Aurora ở Region chính còn hoạt động, vì vậy hãy cân nhắc điều này trong kế hoạch khôi phục thảm họa. Nếu Aurora ở Region chính không khả dụng, bạn vẫn có thể **promote** bất kỳ cluster phụ nào của Aurora trong Aurora global database. Quá trình promotion được thực hiện từ Region phụ. Trong khoảng một phút, quá trình này sẽ loại cluster khỏi Aurora global database, kích hoạt endpoint ghi theo Region, và nâng một instance của cluster lên thành writer chủ động. Khi quá trình này hoàn tất, bạn sẽ cần cập nhật chuỗi kết nối cơ sở dữ liệu trong ứng dụng để kết nối với endpoint writer mới.

Tất cả các cơ sở dữ liệu được thảo luận trong bài viết này đều tuân theo mô hình eventual consistency khi sử dụng đa Region, nhưng các Aurora PostgreSQL–based global database có thêm tùy chọn để chỉ định maximum replica lag cho phép với [managed recovery point objective (managed RPO](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuid…isaster-recovery.html#aurora-global-database-manage-recovery).

Về khả năng backup-and-restore, Aurora có thời gian lưu trữ bản sao tự động tối thiểu 1 ngày và tối đa 35 ngày cho tất cả các cluster, bao gồm cả cluster phụ toàn cầu. Điều này cho phép bạn khôi phục các cluster phụ về một điểm thời gian xác định trong Region đích nếu cần.
Logical replication, sử dụng công nghệ sao chép tích hợp sẵn của engine cơ sở dữ liệu, có thể được thiết lập cho [Amazon RDS](https://aws.amazon.com/rds/) cho MariaDB, MySQL, Oracle, PostgreSQL và Aurora. Một [cross-Region read replica](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.XRgn.html) sẽ nhận và xử lý các thay đổi từ writer ở Region chính. Các replica này cho phép đọc cục bộ nhanh hơn và có thể giảm mất mát dữ liệu cũng như rút ngắn thời gian khôi phục trong trường hợp thảm họa bằng cách được nâng thành instance độc lập. Công nghệ logical replication cũng có thể được dùng để sao chép dữ liệu ra các tài nguyên bên ngoài Amazon RDS, như instance EC2, server on-premises hoặc thậm chí data lake.

Trong những trường hợp RPO và RTO dài hơn vẫn chấp nhận được, các bản sao lưu có thể được sao chép giữa các Region. Điều này áp dụng cho tất cả cơ sở dữ liệu quan hệ và phi quan hệ được nhắc đến trong bài, ngoại trừ ElastiCache. Mặc dù [Amazon Redshift](https://aws.amazon.com/redshift/) không có tính năng sao chép cross-Region, bạn có thể thiết lập Amazon Redshift để [tự động](https://docs.aws.amazon.com/redshift/latest/mgmt/managing-snapshots-console.html#xregioncopy-kms-encrypted-snapshot) sao chép snapshot của kho dữ liệu sang Region khác. Thời gian sao chép bản sao lưu sẽ khác nhau tùy thuộc vào kích thước và tần suất thay đổi dữ liệu.

### Kết hợp lại

Ở cuối mỗi phần trong loạt blog này, chúng tôi xây dựng một ứng dụng mẫu dựa trên các dịch vụ đã đề cập. Điều này cho thấy cách kết hợp các dịch vụ lại với nhau để xây dựng một ứng dụng multi-Region bằng các dịch vụ AWS. Chúng tôi không sử dụng tất cả các dịch vụ được nhắc đến, chỉ những dịch vụ phù hợp với tình huống sử dụng.

Chúng tôi đã tạo ví dụ này để mở rộng ra đối tượng người dùng toàn cầu. Ứng dụng yêu cầu khả năng sẵn sàng cao trên nhiều Region và ưu tiên hiệu năng hơn là tính nhất quán tuyệt đối. Chúng tôi đã chọn những dịch vụ sau (trong số các dịch vụ đã đề cập ở phần này) để đạt được mục tiêu, dựa trên nền tảng từ phần 1:

- DynamoDB global tables để ứng dụng triển khai ở mỗi Region có thể vừa đọc vừa ghi vào cơ sở dữ liệu trong cùng Region.
- ElastiCache Global Datastore làm tầng cache. Việc ghi vào global datastore chỉ diễn ra ở Region 1, nhưng điều này phù hợp với tình huống sử dụng vì ứng dụng dựa vào các mục cache tương tự bất kể Region nào.

- S3 bucket để lưu trữ dữ liệu ứng dụng, tệp cấu hình, nội dung tĩnh và dữ liệu do người dùng tải lên. Amazon S3 sao chép chéo 2 chiều giữa các Region để giữ cho các bucket đồng bộ. Cuối cùng, ứng dụng của chúng tôi sử dụng Amazon S3 multi-Region access point để đảm bảo các yêu cầu đến Amazon S3 được định tuyến đến Region có độ trễ thấp nhất và tự động xử lý failover.

{{< figurecaption src="/images/Img4-Blog6.png" caption="Hình 3. Xây dựng ứng dụng với các dịch vụ AWS multi-Region, sử dụng những dịch vụ đã đề cập trong Phần 1">}}

Mặc dù mục tiêu chính của chúng tôi là mở rộng ra đối tượng toàn cầu, chúng tôi lưu ý rằng một số cơ chế sao chép được thiết lập là một chiều, chẳng hạn như với ElastiCache Global Datastore. Mỗi triển khai theo Region được cấu hình để duy trì tính ổn định tĩnh, nhưng nếu xảy ra sự cố kéo dài ở Region 1, kế hoạch khắc phục thảm họa (DR playbook) của chúng tôi sẽ nêu rõ cách làm cho từng dịch vụ có thể ghi tại Region 2.

### Tóm tắt

Dữ liệu nằm ở trung tâm của hầu hết mọi ứng dụng. Trong bài viết này, chúng tôi đã xem xét các dịch vụ AWS cung cấp khả năng sao chép dữ liệu giữa các Region để đưa dữ liệu của bạn đến đúng nơi một cách nhanh chóng. Dù ứng dụng của bạn cần khả năng đọc cục bộ nhanh hơn, một cơ sở dữ liệu active-active, hay đơn giản chỉ là lưu trữ dữ liệu bền vững ở một Region thứ hai, chúng tôi đều có giải pháp cho bạn. Trong bài viết thứ 3 cũng là cuối cùng của loạt bài này, chúng tôi sẽ đề cập đến các tính năng quản lý và giám sát ứng dụng.