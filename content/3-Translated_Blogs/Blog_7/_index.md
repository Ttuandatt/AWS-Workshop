+++
title = "Blog 7"
weight =  7
chapter = false
pre = " <b> 3.7. </b>"
+++

# Xây dựng Ứng Dụng Đa Vùng với Các Dịch vụ AWS – Phần 3: Quản lý và Giám sát Ứng dụng

*Tác giả: Joe Chapman và Sebastian Leks - 28/03/2022*

*Chủ đề: [AWS CloudFormation](https://aws.amazon.com/blogs/architecture/category/management-tools/aws-cloudformation/), [AWS Config](https://aws.amazon.com/blogs/architecture/category/management-tools/aws-config/)*

---

Trong Phần 1 của loạt bài này, chúng ta đã xây dựng nền tảng cho ứng dụng đa vùng bằng cách sử dụng các dịch vụ [tính toán, mạng và bảo mật](https://aws.amazon.com/blogs/architecture/creating-a-multi-region-application-with-aws-services-part-1-compute-and-security/) của AWS.
Trong Phần 2, chúng ta đã tích hợp các [dịch vụ dữ liệu và sao chép của AWS](https://aws.amazon.com/blogs/architecture/creating-a-multi-region-application-with-aws-services-part-2-data-and-replication/) để di chuyển và đồng bộ dữ liệu giữa các vùng AWS.

Trong Phần 3, chúng ta sẽ tìm hiểu về các dịch vụ và tính năng của AWS được sử dụng cho nhắn tin, triển khai, giám sát và quản lý.

### Công cụ dành cho nhà phát triển

Tự động hóa với cơ sở hạ tầng dưới dạng mã (Infrastructure as Code – IaC) giúp loại bỏ các bước thủ công khi tạo và cấu hình hạ tầng. Nó cung cấp một mẫu có thể tái sử dụng, cho phép triển khai các môi trường nhất quán ở nhiều Vùng khác nhau.

IaC với [AWS CloudFormation StackSets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html) sử dụng một mẫu duy nhất để tạo, cập nhật và xóa [các stack trên nhiều tài khoản và Vùng](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html) chỉ trong một lần thực hiện. Khi viết một mẫu [AWS CloudFormation](http://aws.amazon.com/cloudformation), bạn có thể thay đổi hành vi triển khai bằng cách kết hợp các tham số với logic điều kiện. Ví dụ, bạn có thể đặt một tham số “standby” mà khi được gán giá trị “true” sẽ giới hạn số lượng [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/) instances trong một nhóm [Auto Scaling của Amazon EC2](https://aws.amazon.com/ec2/autoscaling/) được triển khai tại một Vùng dự phòng.

Các ứng dụng có triển khai trải rộng trên nhiều Vùng có thể sử dụng [cross-Region actions](https://docs.aws.amazon.com/codepipeline/latest/userguide/actions-create-cross-region.html) trong [AWS CodePipeline](https://aws.amazon.com/codepipeline/) để xây dựng một pipeline phát hành nhất quán. Cách này giúp bạn không cần phải thiết lập các hành động khác nhau cho từng Vùng. [EC2 Image Builder](https://aws.amazon.com/image-builder) và [Amazon Elastic Container Registry (Amazon ECR)](http://aws.amazon.com/ecr/) có các tính năng sao chép liên vùng (cross-Region copy) để hỗ trợ triển khai AMI và image một cách đồng nhất, như đã đề cập ở Phần 1.

### Kiến trúc hướng sự kiện

Các ứng dụng hướng sự kiện và tách rời (decoupled, event-driven) tạo ra một kiến trúc có khả năng mở rộng và dễ bảo trì hơn, nhờ vào việc mỗi thành phần thực hiện nhiệm vụ cụ thể của nó một cách độc lập.

[Amazon EventBridge](https://aws.amazon.com/eventbridge/), một dịch vụ event bus không máy chủ (serverless), có thể gửi sự kiện giữa các tài nguyên AWS. Bằng cách tận dụng định tuyến sự kiện liên vùng [(cross-Region event routing)](https://aws.amazon.com/blogs/compute/introducing-cross-region-event-routing-with-amazon-eventbridge/), bạn có thể chia sẻ sự kiện giữa các workloads [ở các Vùng](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-cross-region.html) (Hình 1) và [giữa các tài khoản](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-cross-account.html). Ví dụ, bạn có thể chia sẻ các sự kiện về tình trạng và mức sử dụng (health and utilization) giữa các Vùng để xác định workload nào ở Vùng nào là phù hợp nhất để xử lý yêu cầu.

{{< figurecaption src="/images/Img1-Blog7.png" caption="Hình 1. EventBridge định tuyến sự kiện từ một Vùng sang các event bus ở các Vùng khác" >}}

Nếu ứng dụng hướng sự kiện của bạn dựa trên mô hình pub/sub messaging, thì [Amazon Simple Notification Service (Amazon SNS)](https://aws.amazon.com/sns/) có thể phân phối (fan out) thông điệp đến nhiều đích khác nhau.
Khi các đích đến là [Amazon Simple Queue Service (Amazon SQS)](https://aws.amazon.com/sqs/) queues hoặc [AWS Lambda](https://aws.amazon.com/lambda/) functions, Amazon SNS có thể [gửi thông báo đến các đối tượng nhận ở nhiều Vùng khác nhau](https://docs.aws.amazon.com/sns/latest/dg/sns-cross-region-delivery.html).
Ví dụ, bạn có thể gửi thông điệp đến một SQS queue trung tâm để xử lý các đơn hàng cho một ứng dụng đa vùng.

### Giám sát và khả năng quan sát (Monitoring and observability)

Khả năng quan sát (observability) càng trở nên quan trọng khi số lượng tài nguyên và vị trí triển khai tăng lên. Việc có thể nhanh chóng xác định được tác động và nguyên nhân gốc rễ của một sự cố sẽ ảnh hưởng đến các hoạt động khắc phục, và đảm bảo rằng hệ thống quan sát của bạn có khả năng chống chịu với sự cố sẽ giúp bạn đưa ra những quyết định này. Khi xây dựng trên AWS, bạn có thể kết hợp tình trạng sức khỏe của các dịch vụ AWS với các chỉ số ứng dụng của bạn để có được cái nhìn toàn diện hơn về sức khỏe của cơ sở hạ tầng.

AWS Health cung cấp bảng điều khiển (dashboards) và API hiển thị các sự kiện và hoạt động đã lên lịch có thể ảnh hưởng đến tài nguyên của bạn. Những sự kiện này bao phủ tất cả các Region, và có thể mở rộng để bao gồm toàn bộ tài khoản trong AWS Organization. EventBridge có thể giám sát các sự kiện từ AWS Health để thực hiện ngay hành động dựa trên sự kiện đó. Ví dụ, nếu nhiều dịch vụ được báo cáo là đang suy giảm, bạn có thể đặt EventBridge target tới một AWS Systems Manager automated runbook nhằm chuẩn bị ứng dụng disaster recovery (DR) cho việc chuyển đổi dự phòng (failover).

AWS Trusted Advisor cung cấp các cảnh báo mang tính hành động nhằm tối ưu chi phí, tăng hiệu năng, cũng như cải thiện bảo mật và khả năng chịu lỗi. Trusted Advisor hiển thị kết quả trên tất cả các Region và có thể tạo báo cáo tổng hợp kết quả kiểm tra trên toàn bộ tài khoản trong một tổ chức.

Để duy trì khả năng quan sát đối với một ứng dụng được triển khai trên nhiều Region và nhiều tài khoản, bạn có thể tạo Trusted Advisor dashboard và operations dashboard bằng AWS Systems Manager Explorer. Operations dashboard mang đến cái nhìn hợp nhất về các tài nguyên như Amazon EC2, Amazon CloudWatch, và dữ liệu AWS Config. Bạn có thể kết hợp metadata với Amazon Athena để tạo ra một bảng kiểm kê (inventory) đa-Region và đa-tài khoản về các tài nguyên.

Bạn có thể xem các metric từ ứng dụng và tài nguyên được triển khai trên nhiều Region ngay trong CloudWatch console. Điều này giúp dễ dàng tạo biểu đồ và dashboard cho ứng dụng đa-Region. CloudWatch cũng hỗ trợ cross-account, cho phép bạn tạo một cái nhìn tập trung về dashboards, alarms, và metrics trên toàn bộ tổ chức của mình.

Amazon OpenSearch Service tập hợp các log file không cấu trúc hoặc bán cấu trúc, message, metric, document, dữ liệu cấu hình, và nhiều loại dữ liệu khác. Cross-cluster replication (nhân bản chéo cụm) sẽ sao chép indices, mappings, và metadata trong mô hình active-passive từ một domain OpenSearch sang một domain khác. Điều này giúp giảm độ trễ giữa các Region và đảm bảo tính sẵn sàng cao cho dữ liệu.

AWS Resilience Hub đánh giá và theo dõi khả năng chịu lỗi (resiliency) của ứng dụng. Nó kiểm tra mức độ ứng dụng có thể duy trì tính khả dụng khi thực hiện failover ở cấp độ Region. Ví dụ, nó có thể kiểm tra xem ứng dụng đã cấu hình cross-Region replication trên các bucket Amazon S3 chưa, hoặc các instance Amazon RDS đã có read-replica cross-Region hay chưa. Hình 2 minh họa kết quả đánh giá từ Resilience Hub. Nó cũng gợi ý việc sử dụng Route 53 Application Recovery Controller (được đề cập trong Phần 1) để đảm bảo Auto Scaling group của Amazon EC2 trong một Region được scale sẵn và sẵn sàng nhận traffic trước khi chuyển failover sang đó.

{{< figurecaption src="/images/Img2-Blog7.png" caption="Hình 2. Các khuyến nghị của Resilience Hub" >}}

### Quản lý: Quản trị (Governance)
Việc mở rộng một ứng dụng sang một quốc gia mới đồng nghĩa với việc có thể sẽ có thêm những luật và quy định về quyền riêng tư dữ liệu cần tuân thủ. Những điều này sẽ khác nhau tùy theo từng quốc gia, và chúng tôi khuyến nghị bạn nên tham khảo với nhóm pháp lý của mình để hiểu đầy đủ cách chúng ảnh hưởng đến ứng dụng của bạn.

AWS Control Tower hỗ trợ tuân thủ dữ liệu bằng cách cung cấp các guardrails nhằm kiểm soát và đáp ứng các yêu cầu về lưu trú dữ liệu. Các guardrails này là tập hợp của Service Control Policies (SCPs) và các quy tắc AWS Config. Bạn có thể triển khai chúng độc lập với AWS Control Tower nếu cần. Các dịch vụ đa vùng (multi-Region) tập trung vào bảo mật bổ sung đã được đề cập trong Phần 1.

AWS Config cung cấp một cái nhìn chi tiết về cấu hình và lịch sử của các tài nguyên AWS. Một AWS Config aggregator thu thập dữ liệu cấu hình và tuân thủ từ nhiều tài khoản và nhiều Vùng (Region) về một tài khoản trung tâm. Cái nhìn tập trung này mang lại một cái nhìn toàn diện về sự tuân thủ và các hành động trên tài nguyên, bất kể chúng nằm ở tài khoản hay Vùng nào.

### Quản lý: Vận hành (Operations)
Một số khả năng của AWS Systems Manager cho phép quản trị tài nguyên AWS dễ dàng hơn, đặc biệt khi ứng dụng phát triển. Systems Manager Automation đơn giản hóa các tác vụ bảo trì và triển khai phổ biến cho tài nguyên AWS thông qua các runbook tự động. Những runbook này tự động hóa các hành động trên tài nguyên trải rộng nhiều Vùng và nhiều tài khoản.

Bạn có thể kết hợp Systems Manager Automation với Systems Manager Patch Manager để đảm bảo các phiên bản (instances) luôn duy trì bản vá mới nhất trên nhiều tài khoản và nhiều Vùng. Hình 3 cho thấy Systems Manager đang chạy nhiều tài liệu tự động (automation documents) trên một kiến trúc đa vùng.

{{< figurecaption src="/images/Img3-Blog7.png" caption="Hình 3. Sử dụng Systems Manager Automation từ một tài khoản vận hành trung tâm của AWS để tự động hóa các hành động trên nhiều Vùng (Region)." >}}


### Kết hợp lại

Ở cuối mỗi phần của loạt blog này, chúng tôi xây dựng một ứng dụng mẫu dựa trên các dịch vụ đã đề cập. Điều này cho bạn thấy cách kết hợp các dịch vụ để tạo ra một ứng dụng đa vùng (multi-Region) với AWS. Chúng tôi không sử dụng tất cả các dịch vụ đã nhắc đến, mà chỉ chọn những dịch vụ phù hợp với trường hợp sử dụng.

Chúng tôi xây dựng ví dụ này để mở rộng đến một đối tượng toàn cầu. Ứng dụng này yêu cầu tính khả dụng cao trên nhiều vùng và ưu tiên hiệu năng hơn là tính nhất quán nghiêm ngặt. Chúng tôi đã chọn các dịch vụ sau (được đề cập trong bài viết này) để đạt được mục tiêu, đồng thời xây dựng dựa trên nền tảng từ phần 1 và phần 2:

- CloudFormation StackSets để triển khai toàn bộ bằng IaC. Điều này đảm bảo hạ tầng được triển khai nhất quán trên các vùng.

- AWS Config rules cung cấp một nơi tập trung để giám sát, ghi nhận và đánh giá cấu hình tài nguyên của chúng tôi.

- Để tăng khả năng quan sát (observability), chúng tôi đã tạo dashboard với CloudWatch dashboard, Personal Health dashboard, và Trusted Advisor dashboard.

{{< figurecaption src="/images/Img4-Blog7.png" caption="Hình 4. Xây dựng một ứng dụng với các dịch vụ đa vùng (multi-Region)." >}}

Mặc dù mục tiêu chính của chúng tôi là mở rộng đến một đối tượng người dùng toàn cầu, chúng tôi lưu ý rằng một số dịch vụ như CloudFormation StackSets phụ thuộc vào Vùng 1. Mỗi triển khai theo vùng được thiết lập để có tính ổn định tĩnh, nhưng nếu xảy ra sự cố ngừng hoạt động kéo dài ở Vùng 1, kế hoạch khắc phục thảm họa (DR playbook) của chúng tôi sẽ nêu rõ cách thực hiện các thay đổi CloudFormation tại Vùng 2.

### Tóm tắt

Nhiều dịch vụ AWS có các tính năng hỗ trợ bạn xây dựng và quản lý kiến trúc đa vùng (multi-Region), nhưng việc xác định những khả năng này trong hơn 200 dịch vụ có thể gây choáng ngợp.

Trong loạt blog 3 phần này, chúng tôi đã khám phá các dịch vụ AWS với những tính năng giúp bạn xây dựng ứng dụng đa vùng:

- Ở Phần 1, chúng tôi xây dựng nền tảng với các dịch vụ về bảo mật, mạng và điện toán của AWS.

- Ở Phần 2, chúng tôi bổ sung chiến lược dữ liệu và sao chép.

- Cuối cùng, ở Phần 3, chúng tôi xem xét các lớp ứng dụng và quản lý.

Link bài viết gốc: (https://aws.amazon.com/blogs/architecture/creating-a-multi-region-application-with-aws-services-part-3-application-management-and-monitoring/)


Các bài viết khác trong chuỗi này:

- [Xây dựng Ứng dụng Đa Vùng với Các Dịch vụ AWS – Phần 1: Tính toán, Mạng và Bảo mật]({{< relref "3-Translated_Blogs/Blog_5/_index.md" >}})

- [Xây dựng Ứng Dụng Đa Vùng với Các Dịch vụ AWS – Phần 2: Dữ Liệu và Sao Chép]({{< relref "3-Translated_Blogs/Blog_6/_index.md" >}})