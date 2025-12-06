+++
title = "Blog 7"
weight = 7
chapter = false
pre = "<b>3.7.</b>"
+++

# Những hiểu biết sâu sắc và bài học kinh nghiệm từ Amazon Q trong tích hợp trình thu thập thông tin web Connect

**Tác giả:** Vikas Prasad & Ayush Mehta  

**Ngày:** 22/07/2025

## Giới thiệu

Các nhân viên tổng đài (Human agents) là yếu tố then chốt trong mọi trung tâm chăm sóc khách hàng, và việc cung cấp cho họ các công cụ phù hợp để thành công là điều thiết yếu đối với mọi tổ chức.  

Khi được hỗ trợ tốt, các nhân viên không chỉ có trải nghiệm làm việc tốt hơn, mà còn giúp nâng cao trải nghiệm của khách hàng cuối cùng.

Để đáp ứng nhu cầu này, Amazon Connect giới thiệu [Amazon Q in Connect](https://aws.amazon.com/connect/q/) --- một trợ lý AI sinh nội dung (Generative AI assistant), cung cấp các phản hồi và hành động gợi ý theo thời gian thực, được cá nhân hóa giúp nhân viên tổng đài thực hiện công việc hiệu quả và nhanh chóng hơn.

Amazon Q in Connect có thể tích hợp với nhiều cơ sở tri thức (knowledge base) khác nhau thông qua các phương thức tích hợp đa dạng.  

Bài viết này tập trung vào phương pháp tích hợp bằng **Web Crawler**, bao gồm những lưu ý quan trọng khi triển khai và các thực hành tốt nhất.

---

## Vì sao cần tích hợp Web Crawler?

Nội dung thường bị phân tán trên nhiều website, buộc agent phải tra cứu nhiều nguồn khác nhau khi hỗ trợ khách hàng. Điều này làm chậm quá trình xử lý và ảnh hưởng đến chất lượng tương tác.

Khi được cấu hình với **Web Crawler**, Amazon Q in Connect có thể:

- Tự động thu thập và lập chỉ mục nội dung từ nhiều website, trang hỗ trợ  

- Cung cấp cho agent thông tin nhanh chóng và chính xác  

- Hiển thị liên kết nguồn để đối chiếu  

- Loại bỏ việc cập nhật thủ công khi nội dung website thay đổi

![Web crawler phases](/aws/hinhanh/hinh1.jpg)

**Hình 1:** Các giai đoạn tối ưu triển khai Web Crawler

---

## 1. Giai đoạn Planning (Lập kế hoạch)

Trước khi triển khai Web Crawler, bạn nên tham khảo tài liệu chính thức của  

[Amazon Connect -- Enable Amazon Q](https://docs.aws.amazon.com/connect/latest/adminguide/enable-q.html).

Ba nhóm câu hỏi nền tảng cần xác định:

### 1.1. Chiến lược quản lý tri thức

- Ai chịu trách nhiệm quản lý nội dung (Marketing / IT / CX)?

- Quy trình cập nhật nội dung như thế nào?

- Cách đồng bộ nội dung với Amazon Connect

### 1.2. Phân tích người dùng cuối

- Agent tuyến 1 hay hỗ trợ kỹ thuật chuyên sâu?

- Mức độ chi tiết của nội dung cần crawl?

### 1.3. Khả năng truy cập nội dung

- Nội dung công khai hay cần xác thực?

- Tuân thủ bảo mật & quyền riêng tư

---

## 2. Giai đoạn Staged Implementation (Triển khai theo giai đoạn)

- Tạo từng **knowledge base riêng biệt**

- Kiểm thử độc lập các cấu hình URL

- Thu thập phản hồi từ agent

- Mở rộng phạm vi crawl theo từng bước

Cách tiếp cận này giúp:

- Dễ troubleshoot

- Kiểm soát chất lượng dữ liệu

- Tối ưu hiệu suất

---

## 3. Giai đoạn Optimization (Tối ưu hóa)

### 3.1. Giới hạn dịch vụ

- Mỗi crawler tối đa **25.000 file**

- Cần:

  - Ưu tiên nội dung quan trọng

  - Regex URL

  - Metadata filtering

### 3.2. Tốc độ crawl

- Bắt đầu crawl rate thấp

- Theo dõi server response

- Tránh lỗi `504 Gateway Timeout`

### 3.3. Phạm vi crawl

Ví dụ seed URL:

```json

{ "url": "https://example.com/products/" }

{ "url": "https://example.com/documentation/" }

3.4. Chiến lược knowledge base

Unified KB → nhiều sản phẩm

Separate KBs → từng dòng sản phẩm

Có thể dynamic switch bằng:

AI Agent override

Update Session API

4\. Giai đoạn Implementation (Triển khai thực tế)

4.1. Execution Timeout

Ưu tiên URL

Tối ưu crawl pattern

Chia nhỏ crawl job

4.2. URL Configuration

UI: max 10 URLs

API: max 100 URLs

Dùng regex thay vì liệt kê thủ công

Ví dụ:

```regex
.*_domain\.com/support/.*\.pdf
```
Kết luận

Quy trình tích hợp Web Crawler cho Amazon Q in Connect gồm 4 giai đoạn:

Planning

Staged Implementation

Optimization

Implementation

Toàn bộ quy trình mang tính lặp (iterative), liên tục cải thiện dựa trên feedback và metric.

Các lưu ý quan trọng:

Xác định nội dung ưu tiên cao

Đặt seed URL hợp lý

Sử dụng regex để lọc chính xác

Theo dõi qua Amazon CloudWatch Logs

Việc áp dụng đúng chiến lược giúp Amazon Q in Connect trở thành công cụ tri thức mạnh mẽ, hỗ trợ agent hiệu quả và nâng cao trải nghiệm khách hàng.