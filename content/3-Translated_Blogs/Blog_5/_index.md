+++
title = "Blog 5"
weight =  5
chapter = false
pre = " <b> 3.5. </b>"
+++

# Xây dựng Ứng dụng Đa Vùng với Các Dịch vụ AWS – Phần 1: Tính toán, Mạng và Bảo mật

*Tác giả: Joe Chapman và Sebastian Leks - 08/12/2021*

*Chủ đề: [Amazon CloudFront](https://aws.amazon.com/blogs/architecture/category/networking-content-delivery/amazon-cloudfront/), [Amazon EC2](https://aws.amazon.com/blogs/architecture/category/compute/amazon-ec2/), [Amazon Elastic Block Store (Amazon EBS)](https://aws.amazon.com/blogs/architecture/category/storage/amazon-elastic-block-storage-ebs/), [Amazon Route 53](https://aws.amazon.com/blogs/architecture/category/networking-content-delivery/amazon-route-53/), [Amazon Simple Storage Service (S3)](https://aws.amazon.com/blogs/architecture/category/storage/amazon-simple-storage-services-s3/), [Amazon VPC](https://aws.amazon.com/blogs/architecture/category/compute/amazon-vpc/), [Architecture](https://aws.amazon.com/blogs/architecture/category/architecture/), [AWS CloudTrail](https://aws.amazon.com/blogs/architecture/category/management-tools/aws-cloudtrail/), [AWS Global Accelerator](https://aws.amazon.com/blogs/architecture/category/networking-content-delivery/aws-global-accelerator/), [AWS Identity and Access Management (IAM)](https://aws.amazon.com/blogs/architecture/category/security-identity-compliance/aws-identity-and-access-management-iam/), [AWS Secrets Manager](https://aws.amazon.com/blogs/architecture/category/security-identity-compliance/aws-secrets-manager/), [AWS Security Hub](https://aws.amazon.com/blogs/architecture/category/security-identity-compliance/aws-secrets-manager/), [AWS Transit Gateway](https://aws.amazon.com/blogs/architecture/category/security-identity-compliance/aws-secrets-manager/), [AWS Well-Architected Permalink Share](https://aws.amazon.com/blogs/architecture/category/security-identity-compliance/aws-secrets-manager/)*

---

Nhiều dịch vụ AWS có các tính năng giúp bạn xây dựng và quản lý kiến trúc đa vùng (multi-Region), nhưng việc xác định những khả năng này trong hơn 200 dịch vụ có thể là một thách thức lớn.

Trong loạt blog gồm 3 phần này, chúng tôi sẽ chọn lọc từ hơn 200 dịch vụ đó và tập trung vào những dịch vụ có tính năng cụ thể hỗ trợ bạn xây dựng ứng dụng đa vùng. Trong Phần 1, chúng ta sẽ xây dựng nền tảng với các dịch vụ bảo mật, mạng, và tính toán của AWS. Ở Phần 2, chúng ta sẽ bổ sung các chiến lược dữ liệu và sao chép. Cuối cùng, trong Phần 3, chúng ta sẽ tìm hiểu về lớp ứng dụng và quản lý. Khi đi qua từng phần, chúng ta sẽ dần xây dựng một ứng dụng mẫu để minh họa cách kết hợp các dịch vụ này nhằm tạo ra một ứng dụng đa vùng.

### Những điều cần cân nhắc trước khi bắt đầu

Các [AWS Region](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/) được xây dựng với nhiều [Availability Zone (AZ)](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/#Availability_Zones) tách biệt và cách ly về mặt vật lý. Cách tiếp cận này cho phép bạn tạo ra các workload có độ sẵn sàng cao, tuân thủ nguyên tắc [Well-Architected](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html), trải rộng trên nhiều AZ để đạt được khả năng chịu lỗi tốt hơn. Điều này đáp ứng được mục tiêu khả dụng cho hầu hết các ứng dụng, nhưng vẫn có một số lý do chung khiến bạn có thể muốn mở rộng ra ngoài một Region duy nhất:

- Mở rộng ra khán giả toàn cầu: khi ứng dụng phát triển và lượng người dùng trở nên phân tán về mặt địa lý, có thể sẽ cần giảm độ trễ cho các khu vực khác nhau trên thế giới.

- Giảm [RPO (Recovery Point Objective)](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/disaster-recovery-dr-objectives.html) và [RTO (Recovery Time Objective)](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html): như một phần của kế hoạch khắc phục thảm họa (Disaster Recovery – DR) đa vùng.

- Luật pháp và quy định địa phương: có thể có những yêu cầu nghiêm ngặt về lưu trú dữ liệu (data residency) và quyền riêng tư mà bạn buộc phải tuân thủ.

Nếu bạn đang xây dựng một ứng dụng đa vùng mới, bạn nên cân nhắc tập trung vào các dịch vụ AWS có sẵn chức năng hỗ trợ. Với những ứng dụng hiện có, cần phải xem xét kỹ hơn để xác định kiến trúc nào có khả năng mở rộng nhất nhằm đáp ứng sự phát triển. Các phần tiếp theo sẽ xem xét những dịch vụ này, đồng thời nêu bật các trường hợp sử dụng và thực tiễn tốt nhất.

### Quản lý danh tính và truy cập trên nhiều Region

Xây dựng nền tảng bảo mật bắt đầu từ việc thiết lập các quy tắc xác thực (authentication) và phân quyền (authorization) phù hợp. Hệ thống xử lý các yêu cầu này phải có khả năng chịu lỗi cao để xác minh và cấp quyền nhanh chóng, đáng tin cậy. [AWS Identity and Access Management (IAM)](http://aws.amazon.com/iam) đáp ứng điều này bằng cách cung cấp một cơ chế đáng tin cậy để bạn quản lý quyền truy cập vào các dịch vụ và tài nguyên AWS. IAM có khả năng sẵn sàng trên nhiều Region một cách tự động, mà bạn không cần phải cấu hình gì thêm.

Để hỗ trợ quản lý người dùng Windows, thiết bị và ứng dụng trong một mạng đa vùng, bạn có thể thiết lập [AWS Directory Service for Microsoft Active Directory](https://aws.amazon.com/directoryservice/active-directory/) Enterprise Edition, dịch vụ này sẽ [tự động sao chép dữ liệu thư mục giữa các Region](https://docs.aws.amazon.com/directoryservice/latest/admin-guide/ms_ad_configure_multi_region_replication.html). Điều này giúp giảm độ trễ khi tra cứu thư mục bằng cách sử dụng thư mục gần nhất, đồng thời tăng tính bền vững bằng cách trải rộng trên nhiều Region. Lưu ý rằng điều này cũng kéo theo "số phận chung" giữa các domain controller trong kiến trúc đa vùng, vì các thay đổi group policy sẽ được lan truyền đến tất cả các máy chủ thành viên.

Các ứng dụng cần lưu trữ, luân chuyển và kiểm toán bí mật một cách an toàn, chẳng hạn như mật khẩu cơ sở dữ liệu, nên sử dụng [AWS Secrets Manager](https://aws.amazon.com/secrets-manager). Dịch vụ này mã hóa các bí mật bằng khóa của [AWS Key Management Service (AWS KMS)](http://aws.amazon.com/kms) và có thể [sao chép các bí mật sang Region thứ cấp](https://docs.aws.amazon.com/secretsmanager/latest/userguide/create-manage-multi-region-secrets.html) để đảm bảo ứng dụng có thể nhanh chóng lấy được bí mật từ Region gần nhất.

### Mã hóa trên nhiều Region

AWS KMS có thể được sử dụng để mã hóa dữ liệu khi lưu trữ (data at rest) và được dùng rộng rãi cho việc mã hóa trên các dịch vụ AWS. Theo mặc định, các khóa chỉ giới hạn trong một Region. Những dịch vụ AWS như [Amazon Simple Storage Service (Amazon S3) cross-Region replication và Amazon Aurora Global Database](http://aws.amazon.com/s3) (sẽ được đề cập trong [phần 2](https://aws.amazon.com/blogs/architecture/creating-a-multi-region-application-with-aws-services-part-2-data-and-replication/)) giúp đơn giản hóa quá trình mã hóa và giải mã bằng các khóa khác nhau ở từng Region.

Đối với các phần khác trong ứng dụng đa vùng của bạn phụ thuộc vào khóa KMS, bạn có thể thiết lập [AWS KMS multi-Region keys](https://docs.aws.amazon.com/kms/latest/developerguide/multi-region-keys-overview.html) để sao chép cả key material và key ID sang Region thứ hai. Điều này loại bỏ nhu cầu giải mã rồi mã hóa lại dữ liệu với một khóa khác ở từng Region. Ví dụ, multi-Region keys có thể được dùng để giảm độ phức tạp trong các hoạt động mã hóa của ứng dụng đa vùng cho dữ liệu được lưu trữ trên nhiều Region.

### Kiểm toán và khả năng quan sát trên nhiều Region

Một thực tiễn tốt nhất là cấu hình [AWS CloudTrail](http://aws.amazon.com/cloudtrail) để lưu lại toàn bộ hoạt động AWS API liên quan trong tài khoản của bạn nhằm phục vụ mục đích kiểm toán. Khi bạn sử dụng nhiều Region hoặc nhiều tài khoản, các log CloudTrail này nên được [tập hợp](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/receive-cloudtrail-log-files-from-multiple-regions.html) về một bucket Amazon S3 duy nhất để thuận tiện cho việc phân tích. Để ngăn chặn việc sử dụng sai mục đích, các log tập trung này cần được coi là dữ liệu nhạy cảm hơn, chỉ cấp quyền truy cập cho các hệ thống và nhân sự chủ chốt.

Để theo dõi các phát hiện từ [AWS Security Hub](https://aws.amazon.com/security-hub), bạn có thể [tổng hợp và liên kết các phát hiện từ nhiều vị trí về một Region duy nhất](https://docs.aws.amazon.com/securityhub/latest/userguide/finding-aggregation-enable.html). Đây là cách đơn giản để tạo một cái nhìn tập trung về các phát hiện của Security Hub trên nhiều tài khoản và nhiều Region. Sau khi thiết lập, các phát hiện sẽ liên tục được đồng bộ giữa các Region để giúp bạn luôn nắm được kết quả toàn cầu trên một bảng điều khiển duy nhất.

Chúng tôi đã kết hợp những tính năng này trong Hình 1. Chúng tôi sử dụng IAM để cấp quyền truy cập chi tiết đến các dịch vụ và tài nguyên AWS, Directory Service for Microsoft AD để xác thực người dùng trong các ứng dụng Microsoft, và Secrets Manager để lưu trữ thông tin đăng nhập cơ sở dữ liệu nhạy cảm. Dữ liệu của chúng tôi, di chuyển tự do giữa các Region, được mã hóa bằng KMS multi-Region keys, và toàn bộ hoạt động truy cập AWS API được ghi lại bởi CloudTrail, sau đó tập trung vào một bucket S3 trung tâm mà chỉ nhóm bảo mật của chúng tôi mới có quyền truy cập.

{{< figurecaption src="/images/Img1-Blog5.png" caption="Hình 1. Các dịch vụ bảo mật, danh tính và tuân thủ đa vùng" >}}

### Xây dựng mạng toàn cầu

Đối với các tài nguyên được triển khai trong các mạng ảo ở những Region khác nhau, [Amazon Virtual Private Cloud (Amazon VPC)](http://aws.amazon.com/vpc) cho phép [định tuyến riêng tư giữa các Region và tài khoản bằng VPC peering](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html). Các tài nguyên này có thể giao tiếp với nhau bằng địa chỉ IP riêng mà không cần internet gateway, VPN, hoặc thiết bị mạng riêng biệt. Tính năng này hoạt động tốt cho các mạng nhỏ chỉ cần một vài kết nối peering. Tuy nhiên, định tuyến bắc cầu (transitive routing) không được hỗ trợ, và khi số lượng VPC peering tăng lên, cấu trúc mạng dạng mesh có thể trở nên khó quản lý và khắc phục sự cố.

[AWS Transit Gateway](https://aws.amazon.com/transit-gateway/) giúp giảm bớt những khó khăn này bằng cách tạo một hub trung chuyển mạng, kết nối các VPC và mạng tại chỗ (on-premises). Khả năng định tuyến của Transit Gateway có thể mở rộng sang các Region khác thông qua [Transit Gateway inter-Region peering](https://docs.aws.amazon.com/vpc/latest/tgw/tgw-peering.html), để tạo ra một mạng riêng tư phân tán toàn cầu cho tài nguyên của bạn.

Xây dựng một cách định tuyến đáng tin cậy và tiết kiệm chi phí để đưa người dùng đến các ứng dụng internet phân tán đòi hỏi những bản ghi Domain Name System (DNS) có độ sẵn sàng cao và khả năng mở rộng tốt. [Amazon Route 53](http://aws.amazon.com/route53) chính là dịch vụ làm được điều đó.

Route 53 có nhiều [chính sách định tuyến](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html) khác nhau. Ví dụ, bạn có thể định tuyến một yêu cầu đến bản ghi có độ trễ mạng thấp nhất, hoặc đưa người dùng ở một vị trí địa lý cụ thể đến endpoint ứng dụng cục bộ. Đối với kịch bản khắc phục thảm họa (DR), [Route 53 Application Recovery Controller (Route 53 ARC)](https://aws.amazon.com/route53/application-recovery-controller/) cung cấp một giải pháp failover toàn diện với mức phụ thuộc tối thiểu. Các routing policy, safety check, và readiness check của Route 53 ARC giúp bạn thực hiện failover qua nhiều Region, AZ, và môi trường on-premises một cách đáng tin cậy.

[Amazon CloudFront](http://aws.amazon.com/cloudfront) – mạng phân phối nội dung (CDN) – là một dịch vụ toàn cầu, được xây dựng trên hơn 300 điểm hiện diện (PoP) khắp thế giới. Các ứng dụng có nhiều origin khả dụng (ví dụ như nhiều Region) có thể dùng [CloudFront origin failover](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/high_availability_origin_failover.html) để tự động chuyển hướng sang origin dự phòng khi origin chính gặp sự cố. Khả năng của CloudFront không chỉ dừng lại ở việc phân phát nội dung, mà còn có thể chạy tính toán ở edge. [CloudFront Functions](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cloudfront-functions.html) giúp dễ dàng chạy các đoạn mã JavaScript nhẹ, trong khi [AWS Lambda@Edge](https://docs.aws.amazon.com/lambda/latest/dg/lambda-edge.html) cho phép bạn chạy các hàm Node.js và Python gần hơn với người dùng ứng dụng, từ đó cải thiện hiệu năng và giảm độ trễ. Việc đưa compute ra edge giúp giảm tải cho origin và mang lại phản hồi nhanh hơn cho người dùng toàn cầu.

Được xây dựng trên mạng lưới toàn cầu của AWS, [AWS Global Accelerator](http://aws.amazon.com/global-accelerator) cung cấp hai địa chỉ IP anycast tĩnh để làm điểm truy cập duy nhất cho các ứng dụng hướng internet. Bạn có thể linh hoạt thêm hoặc xóa origin trong khi hệ thống vẫn tự động định tuyến lưu lượng đến endpoint khu vực khỏe mạnh gần nhất. Nếu phát hiện lỗi, Global Accelerator sẽ [tự động chuyển hướng lưu lượng đến một endpoint khả dụng](https://docs.aws.amazon.com/global-accelerator/latest/dg/disaster-recovery-resiliency.html) chỉ trong vài giây, mà không cần thay đổi địa chỉ IP tĩnh.

Hình 2 minh họa việc sử dụng Route 53 latency-based routing policy để định tuyến người dùng đến endpoint nhanh nhất, CloudFront để phân phát nội dung tĩnh như video và hình ảnh, và Transit Gateway để tạo một mạng riêng toàn cầu, giúp các thiết bị của chúng tôi có thể giao tiếp an toàn trên nhiều Region.

{{< figurecaption src="/images/Img2-Blog5.png" caption="Hình 2. Kết nối AWS VPC và phân phối nội dung" >}}

### Xây dựng và quản lý lớp tính toán (compute layer)

Mặc dù [Amazon Elastic Compute Cloud (Amazon EC2)](http://aws.amazon.com/ec2) và các [Amazon Elastic Block Store (Amazon EBS)](http://aws.amazon.com/ebs) volume liên kết chỉ tồn tại trong một AZ, [Amazon Data Lifecycle Manager](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/snapshot-lifecycle.html) có thể tự động hóa quá trình tạo và sao chép snapshot EBS giữa các Region. Điều này giúp nâng cao chiến lược khắc phục thảm họa (DR) bằng cách cung cấp một lựa chọn sao lưu và khôi phục lạnh (cold backup-and-restore) đơn giản cho các volume EBS. Nếu bạn cần sao lưu nhiều hơn chỉ các EBS volume, [AWS Backup](https://aws.amazon.com/backup/) cung cấp một nơi tập trung để thực hiện việc này trên nhiều dịch vụ (sẽ được đề cập trong [phần 2](https://aws.amazon.com/blogs/architecture/creating-a-multi-region-application-with-aws-services-part-2-data-and-replication/)).

Một EC2 instance được xây dựng dựa trên một [Amazon Machine Image (AMI)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html). AMI xác định cấu hình của instance như lưu trữ, quyền khởi chạy, và ánh xạ thiết bị. Khi cần tạo và phát hành một image chuẩn mới, [EC2 Image Builder](https://aws.amazon.com/image-builder/) giúp đơn giản hóa quá trình xây dựng, kiểm thử, và triển khai AMI mới. Nó cũng hỗ trợ sao chép AMI sang các Region bổ sung, loại bỏ việc phải sao chép thủ công AMI nguồn sang các Region đích.

Các ứng dụng dựa trên microservice sử dụng container sẽ hưởng lợi từ thời gian khởi động nhanh hơn. [Amazon Elastic Container Registry (Amazon ECR)](http://aws.amazon.com/ecr/) có thể đảm bảo điều này diễn ra nhất quán trên nhiều Region bằng cách sao chép image riêng tư ở cấp độ registry. Một ECR private registry có thể được cấu hình cho cả cross-Region hoặc cross-account replication, để đảm bảo image của bạn luôn sẵn sàng ở các Region thứ cấp khi cần.

Khi kiến trúc mở rộng ra nhiều Region, việc theo dõi tài nguyên được cấp phát ở đâu có thể trở nên khó khăn. [Amazon EC2 Global View](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Filtering.html#global-view) giúp giảm bớt vấn đề này bằng cách cung cấp một bảng điều khiển tập trung, hiển thị các tài nguyên EC2 như instance, VPC, subnet, security group, và volume trong tất cả các Region đang hoạt động.

Chúng tôi kết hợp các tính năng compute layer này trong Hình 3 bằng cách sử dụng EC2 Image Builder để sao chép AMI chuẩn mới nhất của chúng tôi sang nhiều Region để triển khai. Chúng tôi cũng sao lưu mỗi EBS volume trong 3 ngày và sao chép nó sang nhiều Region bằng Data Lifecycle Manager.

{{< figurecaption src="/images/Img3-Blog5.png" caption="Hình 3. Sao chép AMI và ảnh chụp nhanh EBS giữa các Vùng (Regions)" >}}


### Đưa tất cả lại với nhau

Ở cuối mỗi phần của loạt bài blog này, chúng tôi sẽ xây dựng một ứng dụng mẫu dựa trên các dịch vụ đã đề cập. Điều này cho thấy cách bạn có thể kết hợp các dịch vụ để xây dựng một ứng dụng đa Vùng (multi-Region) với AWS. Chúng tôi không sử dụng tất cả dịch vụ được nhắc đến, chỉ chọn những dịch vụ phù hợp với trường hợp sử dụng.

Chúng tôi xây dựng ví dụ này để mở rộng đến phạm vi toàn cầu. Ứng dụng yêu cầu tính sẵn sàng cao giữa các Vùng, và ưu tiên hiệu năng hơn là tính nhất quán tuyệt đối. Chúng tôi đã chọn các dịch vụ sau (trong bài viết này) để đạt được mục tiêu:

- Route 53 với chính sách định tuyến theo độ trễ (latency routing) để đưa người dùng đến vùng triển khai có độ trễ thấp nhất.

- CloudFront được thiết lập để phân phối nội dung tĩnh. Region 1 là nguồn gốc chính, nhưng chúng tôi đã cấu hình dự phòng nguồn gốc (origin failover) sang Region 2 trong trường hợp có sự cố.

- Ứng dụng phụ thuộc vào một số API của bên thứ ba, vì vậy Secrets Manager với khả năng sao chép đa Vùng đã được thiết lập để lưu trữ thông tin khóa API nhạy cảm.

- CloudTrail logs được tập trung tại Region 1 để dễ dàng phân tích và kiểm toán.

- Security Hub tại Region 1 được chọn làm nơi tập hợp các phát hiện từ tất cả các Vùng.

- Đây là ứng dụng dựa trên container, chúng tôi dựa vào Amazon ECR replication tại mỗi vị trí để nhanh chóng tải về các image mới nhất tại chỗ.

- Để liên lạc qua IP riêng giữa các Vùng, một Transit Gateway được thiết lập tại mỗi Vùng với kết nối liên Vùng. VPC peering cũng có thể hoạt động, nhưng vì dự kiến mở rộng ra nhiều Vùng hơn trong tương lai nên chúng tôi chọn Transit Gateway như giải pháp lâu dài.

- IAM được dùng để cấp quyền quản lý tài nguyên AWS.

{{< figurecaption src="/images/Img4-Blog5.png" caption="Hình 4. Xây dựng ứng dụng với các dịch vụ AWS đa Vùng, sử dụng những dịch vụ đã đề cập trong Phần 1" >}}


### Tóm tắt

Khi thiết kế một ứng dụng đa Vùng (multi-Region), việc xây dựng một nền tảng vững chắc là vô cùng quan trọng. Nền tảng này sẽ giúp bạn phát triển ứng dụng nhanh chóng theo cách an toàn, đáng tin cậy và linh hoạt. Nhiều dịch vụ AWS đã tích hợp sẵn các tính năng hỗ trợ bạn xây dựng kiến trúc đa Vùng.

Tùy vào lý do mở rộng ra ngoài một Vùng duy nhất mà kiến trúc của bạn sẽ khác nhau. Trong bài viết này, chúng tôi đã đề cập đến các tính năng cụ thể của những dịch vụ AWS về bảo mật, mạng và tính toán (compute) — với khả năng tích hợp sẵn để giảm bớt khối lượng công việc nặng nề và lặp lại.

Trong các bài viết tiếp theo, chúng tôi sẽ tiếp tục đề cập đến các dịch vụ về dữ liệu, ứng dụng và quản lý.

*** Sẵn sàng để bắt đầu? ***
Chúng tôi đã chọn một số [AWS Solutions](https://aws.amazon.com/solutions/) và [AWS Blogs](https://aws.amazon.com/blogs/?awsf.blog-master-category=*all&awsf.blog-master-learning-levels=*all&awsf.blog-master-industry=*all&awsf.blog-master-analytics-products=*all&awsf.blog-master-artificial-intelligence=*all&awsf.blog-master-aws-cloud-financial-management=*all&awsf.blog-master-business-applications=*all&awsf.blog-master-compute=*all&awsf.blog-master-customer-enablement=*all&awsf.blog-master-customer-engagement=*all&awsf.blog-master-database=*all&awsf.blog-master-developer-tools=*all&awsf.blog-master-devops=*all&awsf.blog-master-end-user-computing=*all&awsf.blog-master-mobile=*all&awsf.blog-master-iot=*all&awsf.blog-master-management-governance=*all&awsf.blog-master-media-services=*all&awsf.blog-master-migration-transfer=*all&awsf.blog-master-migration-solutions=*all&awsf.blog-master-networking-content-delivery=*all&awsf.blog-master-programming-language=*all&awsf.blog-master-sector=*all&awsf.blog-master-security=*all&awsf.blog-master-storage=*all&filtered-posts.q=multi-region&filtered-posts.q_operator=AND) để hỗ trợ bạn!

*** Bạn đang tìm thêm nội dung về kiến trúc? ***
 [AWS Architecture Center](https://aws.amazon.com/architecture/) cung cấp sơ đồ kiến trúc tham chiếu, các giải pháp kiến trúc đã được kiểm chứng, những thực tiễn tốt nhất theo Well-Architected, các mẫu (patterns), biểu tượng và nhiều hơn nữa!