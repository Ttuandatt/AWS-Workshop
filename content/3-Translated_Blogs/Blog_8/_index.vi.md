+++
title = "Blog 8"
weight = 8
chapter = false
pre = "<b> 3.8. </b>"
+++

# Xây dựng hệ thống đa-tenant resilient với Amazon SQS Fair Queues

**Tác giả:** Maximilian Schellhorn & Dirk Fröhner  
**Ngày:** 21/07/2025  
**Chuyên mục:** Amazon Simple Queue Service (SQS), Announcements, Intermediate (200), Serverless, Technical How-to  

---

## Giới thiệu

AWS chính thức giới thiệu **Amazon SQS Fair Queues** — một tính năng mới giúp giảm thiểu hiện tượng *noisy neighbor* trong các hệ thống **multi-tenant**. Với fair queues, ứng dụng trở nên resilient hơn, dễ vận hành hơn, đồng thời giảm chi phí vận hành và cải thiện chất lượng dịch vụ cho khách hàng.

Trong kiến trúc phân tán, **message queue** đóng vai trò nền tảng để xây dựng các hệ thống có khả năng chịu lỗi cao. Queue hoạt động như một lớp đệm (buffer) giữa các thành phần, cho phép xử lý bất đồng bộ và theo tốc độ riêng của từng service. Khi hệ thống gặp lượng truy cập tăng đột biến, queue giúp ngăn lỗi lan truyền bằng cách giữ lại các task và bảo vệ các service phía sau khỏi quá tải.

Amazon SQS từ lâu đã là lựa chọn phổ biến để xây dựng ứng dụng có khả năng mở rộng, nhờ là một **dịch vụ serverless được quản lý hoàn toàn**, có thể xử lý hàng triệu message mỗi giây. Trong bài viết này, bạn sẽ tìm hiểu cách hoạt động của Amazon SQS Fair Queues và cách áp dụng chúng trong hệ thống thực tế.

---

## Tổng quan về bài toán multi-tenant

Nhiều ứng dụng hiện đại áp dụng kiến trúc **multi-tenant**, trong đó một phiên bản ứng dụng phục vụ nhiều tenant khác nhau. Tenant có thể là khách hàng, ứng dụng client, hoặc một loại request cụ thể. Mô hình này giúp:
- Tối ưu chi phí vận hành  
- Đơn giản hóa bảo trì  
- Tăng hiệu quả sử dụng tài nguyên dùng chung  

Tuy nhiên, kiến trúc multi-tenant đối mặt với vấn đề **noisy neighbor** — khi một tenant tiêu thụ quá nhiều tài nguyên và ảnh hưởng đến các tenant khác. Trong hệ thống dựa trên queue, tenant này có thể:
- Gửi lượng message lớn
- Có message mất nhiều thời gian xử lý

Điều này làm tăng **dwell time** (thời gian message nằm trong queue) cho tất cả tenant, gây suy giảm chất lượng dịch vụ và buộc hệ thống phải scale dư thừa hoặc xây dựng logic tùy chỉnh phức tạp.

**Amazon SQS Fair Queues** giải quyết vấn đề này bằng cách đảm bảo các tenant “yên lặng” vẫn được phục vụ công bằng, mà **không cần thay đổi logic xử lý message hiện có** ở consumer.

---

## Cách Amazon SQS Fair Queues hoạt động

Amazon SQS liên tục theo dõi số lượng message **in-flight** (đã nhận nhưng chưa bị xóa) của từng tenant. Khi phát hiện mất cân bằng, hệ thống sẽ:

- Nhận diện tenant gây ồn (noisy tenant)
- Ưu tiên phân phối message của các tenant yên lặng
- Duy trì tổng throughput của queue

### Trạng thái steady state

Khi không có backlog, message được phân phối đều giữa các tenant. Dwell time thấp cho tất cả tenant.

![Hình 1: Multi-tenant queue ở trạng thái steady state](/aws/hinhanh/hinh3.jpg)

---

### Khi xuất hiện noisy tenant

Tenant A gửi lượng lớn message, tạo backlog. Consumer chủ yếu xử lý message của tenant A, khiến dwell time của các tenant khác tăng lên.

![Hình 2: Multi-tenant queue với noisy tenant](/aws/hinhanh/hinh4.jpg)

---

### Khi kích hoạt Fair Queues

SQS xác định tenant A là noisy neighbor và ưu tiên message của các tenant B, C, D. Dwell time của các tenant yên lặng được duy trì thấp, trong khi tenant gây ồn chấp nhận có thời gian chờ cao hơn mà không ảnh hưởng đến tenant khác.

![Hình 3: Multi-tenant queue với SQS Fair Queues](/aws/hinhanh/hinh5.jpg)

**Lưu ý:**

- Fair queues **không giới hạn throughput theo tenant**
- Consumer vẫn xử lý message từ noisy tenant khi còn tài nguyên
- Không giới hạn số lượng tenant
- Không ảnh hưởng độ trễ API

---

## Cách sử dụng Amazon SQS Fair Queues

### 1. Kích hoạt Fair Queues bằng MessageGroupId

Để sử dụng Fair Queues:

1. Producer gửi message kèm **MessageGroupId** (định danh tenant)
2. Cấu hình CloudWatch để theo dõi metric
3. Quan sát hành vi queue với workload khác nhau

![Thêm MessageGroupId vào message](/aws/hinhanh/hinh6.jpg)

Amazon SQS sẽ **tự động kích hoạt Fair Queues** cho các SQS Standard Queue chứa MessageGroupId:
- Không cần thay đổi code consumer
- Không giới hạn throughput
- Không tăng latency API

---

### 2. Theo dõi bằng Amazon CloudWatch Metrics

Amazon SQS cung cấp các metric mới để theo dõi Fair Queues:

- `ApproximateNumberOfNoisyGroups`
- `ApproximateNumberOfMessagesVisibleInQuietGroups`
- `ApproximateAgeOfOldestMessageInQuietGroups`
- `ApproximateNumberOfMessagesNotVisibleInQuietGroups`
- `ApproximateNumberOfMessagesDelayedInQuietGroups`

Metric quan trọng nhất là:

**`ApproximateNumberOfNoisyGroups`**  
→ Giúp phát hiện tenant tiêu thụ tài nguyên quá mức và thiết lập cảnh báo.

Các metric với hậu tố **InQuietGroups** cho phép theo dõi riêng các tenant không gây ồn, thay vì toàn queue.

---

## Theo dõi hiệu ứng công bằng

So sánh metric **InQuietGroups** với metric queue thông thường:

- Backlog toàn queue tăng khi có noisy tenant
- Metric của quiet groups vẫn giữ ở mức thấp
- Chứng tỏ tenant khác không bị ảnh hưởng

![Hình 4: So sánh backlog noisy vs quiet tenant](/aws/hinhanh/hinh7.jpg)

---

## Xác định tenant gây tải cao

Sử dụng **Amazon CloudWatch Contributor Insights** để:
- Xác định top-N tenant tiêu thụ nhiều tài nguyên
- Theo dõi tổng số tenant
- Tránh chi phí metric cao do high-cardinality

![Hình 5: Contributor Insights theo MessageGroupId](/aws/hinhanh/hinh8.jpg)

Contributor Insights tạo metric từ log ứng dụng, do đó ứng dụng cần log số lượng message và MessageGroupId.

---

## Ứng dụng ví dụ

AWS cung cấp một **sample application** để minh họa Amazon SQS Fair Queues:

- Load generator mô phỏng traffic multi-tenant
- CloudWatch dashboard trực quan hóa các metric quan trọng
- Infrastructure as Code (IaC) đầy đủ

![Hình 6: CloudWatch FairQueues Dashboard](/aws/hinhanh/hinh9.jpg)

Mã nguồn và hướng dẫn chạy:
👉 https://github.com/aws-samples/sqs-fair-queues

---

## Kết luận

Amazon SQS Fair Queues tự động giảm thiểu ảnh hưởng của noisy neighbor trong các hệ thống multi-tenant. Chỉ cần thêm định danh tenant vào message, Amazon SQS sẽ tự động:
- Phát hiện tenant gây ồn
- Bảo vệ tenant khác khỏi bị ảnh hưởng
- Duy trì throughput và độ trễ ổn định

Tính năng này giúp xây dựng hệ thống **resilient, dễ vận hành và tiết kiệm chi phí**, đặc biệt phù hợp cho các kiến trúc serverless và event-driven.

---

## Tài liệu tham khảo

- Amazon SQS Developer Guide  
- Amazon SQS Fair Queues Documentation  
- AWS Official Blog