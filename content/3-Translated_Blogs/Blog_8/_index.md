+++
title = "Blog 8"
weight =  8
chapter = false
pre = " <b> 3.8. </b>"
+++

# Xây dựng hệ thống đa tenant resilient với hàng đợi công bằng Amazon SQS

**Tác giả:** Maximilian Schellhorn & Dirk Fröhner  
**Ngày:** 21/07/2025  
**Chuyên mục:** [Amazon Simple Queue Service (SQS)](https://aws.amazon.com/blogs/compute/category/messaging/amazon-simple-queue-service-sqs/), [Announcements](https://aws.amazon.com/blogs/compute/category/post-types/announcements/), [Intermediate (200)](https://aws.amazon.com/blogs/compute/category/learning-levels/intermediate-200/), [Serverless](https://aws.amazon.com/blogs/compute/category/serverless/), [Technical How-to](https://aws.amazon.com/blogs/compute/category/post-types/technical-how-to/)

---

## Giới thiệu

Hôm nay, AWS giới thiệu [Amazon Simple Queue Service (Amazon SQS)](https://aws.amazon.com/sqs/) fair queues — một tính năng mới giúp giảm thiểu ảnh hưởng “[noisy neighbor](https://docs.aws.amazon.com/wellarchitected/latest/saas-lens/noisy-neighbor.html)” trong các hệ thống đa tenant. Với fair queues, ứng dụng của bạn trở nên resilient hơn và dễ vận hành hơn, giảm chi phí vận hành trong khi cải thiện chất lượng dịch vụ cho khách hàng.

Trong kiến trúc phân tán, hàng đợi thông điệp (message queue) đã trở thành xương sống của thiết kế hệ thống resilient. Chúng hoạt động như bộ đệm giữa các thành phần, cho phép dịch vụ xử lý công việc bất đồng bộ và theo tốc độ riêng của chúng. Khi ứng dụng của bạn bất ngờ gặp lượng truy cập tăng đột biến, hàng đợi sẽ ngăn việc lỗi lan truyền bằng cách buffer công việc và đảm bảo các service phía sau không bị quá tải.

Amazon SQS từ lâu đã là lựa chọn hàng đầu cho các nhà phát triển xây dựng ứng dụng có khả năng mở rộng bởi vì đây là dịch vụ serverless được quản lý hoàn toàn, có thể mở rộng liền mạch để tiếp nhận hàng triệu message mỗi giây.
Trong bài viết này, bạn sẽ học cách sử dụng Amazon SQS fair queues và hiểu cơ chế hoạt động của chúng thông qua một ví dụ thực tế.

---

## Tổng quan

Nhiều ứng dụng hiện đại sử dụng kiến trúc multi-tenant, nơi một phiên bản ứng dụng duy nhất phục vụ nhiều tenant. Tenant là bất kỳ thực thể nào chia sẻ tài nguyên với thực thể khác. Đó có thể là khách hàng, ứng dụng khách, hoặc loại yêu cầu. Cách tiếp cận này giúp giảm chi phí vận hành và đơn giản hóa việc bảo trì thông qua việc sử dụng tài nguyên hiệu quả. Một ví dụ về tài nguyên được chia sẻ như vậy là các hàng đợi và năng lực xử lý (consumer capacity) tương ứng của chúng.

Tuy nhiên, hệ thống multi-tenant gặp thách thức khi một tenant trở thành “noisy neighbor” — nghĩa là tenant này làm ảnh hưởng đến những tenant khác bằng cách sử dụng quá mức tài nguyên của hệ thống. Với hàng đợi, tenant này có thể tạo backlog bằng việc gửi lượng lớn message hoặc yêu cầu thời gian xử lý lâu hơn. Hàng đợi thông thường sẽ xử lý các message cũ hơn trước, điều này làm tăng thời gian lưu message (dwell time) cho tất cả tenant trong tình huống như vậy. Điều này khiến việc duy trì chất lượng dịch vụ trở nên khó khăn và buộc đội ngũ phải mở rộng tài nguyên quá mức hoặc xây dựng các giải pháp tùy chỉnh phức tạp.

Amazon SQS fair queues giúp duy trì thời gian lưu message thấp cho các tenant khác khi có một “noisy neighbor”. Việc này diễn ra một cách minh bạch mà không yêu cầu thay đổi logic xử lý message hiện tại của bạn. Bạn chỉ cần định nghĩa tenant trong hệ thống của mình là gì, và Amazon SQS sẽ xử lý điều phối phức tạp để giảm thiểu ảnh hưởng của noisy neighbor.

---

## Cách hoạt động

Amazon SQS liên tục theo dõi sự phân bố các message đã được nhận nhưng chưa bị xóa (in-flight) bởi các consumer trên tất cả tenant. Khi hệ thống phát hiện sự mất cân bằng:

- Nó xác định tenant gây ồn — tenant khiến hàng đợi bị backlog.

- Nó tự động điều chỉnh thứ tự phân phối message để ưu tiên message của các tenant “yên lặng” (không gây ồn).

- Nó duy trì tổng throughput của hàng đợi.

Hãy xem ví dụ sau với một hàng đợi multi-tenant và bốn tenant khác nhau (A, B, C và D).
Trong trạng thái ổn định (steady state), hàng đợi không có backlog, và message in-flight được phân phối đều giữa các tenant. Tất cả message được xử lý ngay khi xuất hiện. Thời gian lưu message (dwell time) thấp cho tất cả tenant. Lưu ý rằng năng lực xử lý của consumer không phải lúc nào cũng được dùng hết trong trạng thái này.

![alt text](/aws/hinhanh/hinh3.jpg)

**Hình 1: Một hàng đợi multi-tenant ở trạng thái steady state**

Bây giờ xét kịch bản có tenant gây ồn: số lượng message của tenant A tăng mạnh và tạo backlog trong hàng đợi. Consumer bận xử lý chủ yếu message từ tenant A, còn các message từ các tenant khác phải chờ trong hàng đợi, dẫn tới tăng dwell time cho tất cả.

![alt text](/aws/hinhanh/hinh4.jpg)

**Hình 2: Hàng đợi multi-tenant với một noisy tenant**

Khi một tenant bắt đầu chiếm phần lớn tài nguyên consumer, Amazon SQS fair queues xem tenant đó là noisy neighbor và ưu tiên trả message của các tenant còn lại. Cách ưu tiên này giúp duy trì dwell time thấp cho các tenant yên lặng (B, C, D), trong khi dwell time của tenant A sẽ cao hơn cho tới khi backlog của nó được xử lý — nhưng không ảnh hưởng đến các tenant khác.

![alt text](/aws/hinhanh/hinh5.jpg)

**Hình 3: Hàng đợi multi-tenant với fair queues**

Amazon SQS không giới hạn tốc độ tiêu thụ theo tenant. Consumer vẫn có thể nhận message từ tenant gây ồn khi còn tài nguyên xử lý và hàng đợi không còn message nào khác để trả về. Giống như SQS Standard queue, fair queues cho phép throughput gần như không giới hạn, và không có giới hạn số lượng tenant trong hàng đợi.

## Cách sử dụng

### 1. Kích hoạt fair queues với MessageGroupId

Dưới đây là phần giới thiệu nhanh về cách bắt đầu sử dụng Amazon SQS fair queues trong ứng dụng của bạn. [Xem tài liệu tính năng](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-fair-queues.html) để có hướng dẫn chi tiết. Đây là các bước chính:

1. Kích hoạt Amazon SQS fair queues bằng cách thêm định danh tenant (MessageGroupId) vào message
2. Cấu hình [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) metrics để theo dõi hoạt động của SQS fair queues

3. Bạn có thể dùng ứng dụng mẫu để quan sát hành vi của fair queues với khối lượng message thay đổi

Kích hoạt Amazon SQS fair queues bằng cách thêm MessageGroupId (tenant identifier)
Producer có thể thêm định danh tenant bằng cách đặt MessageGroupId cho message gửi đi:
![alt text](/aws/hinhanh/hinh6.jpg)

Tính năng fairness mới sẽ được áp dụng tự động cho tất cả SQS standard queues đối với các message có thuộc tính MessageGroupId.
Không cần thay đổi code phía consumer.
Không ảnh hưởng đến độ trễ API và không có giới hạn throughput.

### Cấu hình Amazon CloudWatch Metrics để theo dõi SQS Fair Queues

Bạn có thể giám sát SQS fair queues bằng [CloudWatch metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html). Các thuật ngữ quan trọng:

- Noisy groups – nhóm message thuộc tenant gây ồn

- Quiet groups – các nhóm message còn lại (tenant không gây ồn)

SQS giờ cung cấp một số metric mới:

- ApproximateNumberOfNoisyGroups

- ApproximateNumberOfMessagesVisibleInQuietGroups

- ApproximateNumberOfMessagesNotVisibleInQuietGroups

- ApproximateNumberOfMessagesDelayedInQuietGroups

- ApproximateAgeOfOldestMessageInQuietGroups

Metric mới quan trọng nhất là:
ApproximateNumberOfNoisyGroups — số group (tenant) được coi là noisy.
Dùng nó để phát hiện tenant tiêu thụ tài nguyên quá mức và thiết lập cảnh báo.
Fair queues cung cấp thêm các metric với hậu tố InQuietGroups, cho phép theo dõi riêng các tenant không gây ồn — thay vì toàn hàng đợi.

## Theo dõi hiệu ứng công bằng

Bạn có thể **so sánh metric `InQuietGroups` với metric queue thông thường**:

- Khi một tenant tạo traffic đột biến, metric toàn queue tăng backlog
- Nhưng metric quiet groups vẫn thấp → chứng tỏ tenant khác không bị ảnh hưởng

![alt text](/aws/hinhanh/hinh7.jpg)

**Hình 4:** Biểu đồ backlog tenant noisy vs quiet

---

## Xác định tenant gây tải cao

Dùng **[Amazon CloudWatch Contributor Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContributorInsights.html)** để:

- Xem top-N tenant
- Tổng số tenant
- Mức độ sử dụng của từng tenant

> Điều này hữu ích khi có hàng nghìn tenant, giúp tránh chi phí metric lớn (**high-cardinality**).

![alt text](/aws/hinhanh/hinh8.jpg)

**Hình 5:** Dashboard Contributor Insights dựa trên MessageGroupId

Contributor Insights tạo metric từ log ứng dụng của bạn.
Ứng dụng cần log số lượng message xử lý và MessageGroupId.
Ví dụ đầy đủ có trong sample app phần tiếp theo.

---

## Ứng dụng ví dụ

Để giúp bạn bắt đầu dễ dàng hơn, chúng tôi đã chuẩn bị một ứng dụng mẫu để bạn có thể quan sát hành vi của Amazon SQS fair queues với các mức tải message khác nhau. Bạn có thể tìm mã nguồn, hạ tầng dưới dạng code (IaC) và [hướng dẫn chạy mẫu](https://github.com/aws-samples/sample-amazon-sqs-fair-queues) trong kho GitHub sqs-fair-queues.

Ứng dụng ví dụ này bao gồm một trình tạo tải (load generator) để mô phỏng traffic multi-tenant và cung cấp một [CloudWatch dashboard](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Dashboards.html) hiển thị các metric quan trọng nhất để trực quan hóa cách fair queues hoạt động. Hình dưới đây minh hoạ dashboard đó:

![alt text](/aws/hinhanh/hinh9.jpg)

**Hình 6:** CloudWatch FairQueuesDashboard

- Mã nguồn và hướng dẫn chạy trong GitHub: [sqs-fair-queues](https://github.com/aws-samples/sqs-fair-queues)

---

## Kết luận

Amazon SQS fair queues tự động giảm thiểu ảnh hưởng của noisy neighbor trong các hàng đợi multi-tenant. Ngay cả khi một tenant tạo ra lượng message lớn hoặc cần thời gian xử lý lâu hơn (tức trở thành noisy neighbor), tính năng này vẫn duy trì thời gian lưu message ổn định cho các tenant khác. Khi bạn thêm định danh tenant vào message, Amazon SQS fair queues sẽ tự động phát hiện và giảm thiểu tác động của tenant gây ồn, đảm bảo quyền truy cập công bằng vào hàng đợi cho các tenant còn lại.

Chúng tôi khuyến nghị bạn xem [Amazon SQS Developer Guide](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html) để bắt đầu và thử nghiệm với ứng dụng mẫu nhằm quan sát hành vi của hệ thống với các mức tải khác nhau.

---

## Tham khảo

- [Amazon SQS Developer Guide](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html)
- [AWS Blog – Fair Queues](https://aws.amazon.com/blogs/)
