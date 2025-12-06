+++

title = "Blog 12"

weight = 12

chapter = false

pre = "<b> 3.12. </b>"

+++

# Hỗ trợ tùy chỉnh trên quy mô lớn: Chuyển đổi một cơ sở kiến thức Salesforce hợp nhất thành các tác tử AI theo từng LOB

**Tác giả:**  

Bhaskar Rao, Saqib M, Dipkumar Mehta, Murtuza Kainan

**Ngày:** 01/08/2025

**Danh mục:** Nâng cao (300), Amazon AppFlow, Amazon Connect, Amazon Q, Customer Engagement, Customer Solutions, Technical Guide

---

## Giới thiệu

Trong bối cảnh các hệ thống CRM ngày càng phức tạp, các nhóm hỗ trợ khách hàng thường phải vận hành nhiều sản phẩm và dịch vụ khác nhau trong cùng một nền tảng như **Salesforce**. Tuy nhiên, việc sử dụng một **cơ sở kiến thức (Knowledge Base -- KB) hợp nhất** cho tất cả các ngành kinh doanh (LOB -- Line of Business) khiến nhân viên hỗ trợ mất nhiều thời gian tìm kiếm thông tin liên quan.

Điều này dẫn đến:

- Tăng thời gian xử lý cuộc gọi  

- Giảm tỷ lệ giải quyết ngay lần đầu  

- Trải nghiệm khách hàng kém

Bài blog này trình bày một cách tiếp cận **mở rộng và tối ưu hóa hệ thống kiến thức** bằng cách phân tách KB theo từng LOB, kết hợp với **Amazon Q in Connect** và **Salesforce Knowledge**.

---

## Thách thức & Giải pháp

### Thách thức

Khi Salesforce Knowledge được đồng bộ trực tiếp vào Amazon Q in Connect, toàn bộ nội dung được lưu trong **một kho hợp nhất**. Cách tiếp cận này không phù hợp với các tổ chức đa ngành vì:

- Nội dung bị nhiễu

- Kết quả tìm kiếm kém liên quan

- Hiệu suất AI không tối ưu

### Giải pháp

Sử dụng **AI Agents trong Amazon Q in Connect** để:

- Phân tách Salesforce Knowledge thành **nhiều KB theo từng LOB**

- Áp dụng **AI Prompts chuyên biệt** cho từng lĩnh vực

- Giữ nguyên tích hợp Salesforce, nhưng triển khai logic KB tách biệt

**Lợi ích chính:**

- Kết quả tìm kiếm chính xác hơn  

- Thời gian xử lý nhanh hơn  

- Trải nghiệm nhân viên và khách hàng tốt hơn

---

## Nội dung bạn sẽ học được

Thông qua blog này, bạn sẽ:

- Hiểu kiến trúc tích hợp Salesforce → Amazon Q in Connect

- Tạo nhiều KB theo LOB từ Salesforce Knowledge

- Tự động hóa nhập dữ liệu và phân loại KB

- Triển khai AI Agents và AI Prompts theo từng LOB

- Đo lường hiệu quả hệ thống sau triển khai

---

## Đo lường thành công

**Các chỉ số đánh giá:**

- Giảm thời gian tìm kiếm thông tin của nhân viên  

- Tăng tỷ lệ First Call Resolution (FCR)  

- Nâng cao mức độ hài lòng của khách hàng  

- Dashboard và best practices báo cáo rõ ràng

---

## Tổng quan kiến trúc giải pháp

_(Hình minh họa kiến trúc tổng thể)_

Kiến trúc chia Salesforce Knowledge theo từng Business Unit (BU), xử lý dữ liệu qua AppFlow, S3, SQS, Lambda và đồng bộ vào Amazon Q in Connect.

---

# Hướng dẫn triển khai

## A. Tích hợp Salesforce với AWS AppFlow

Salesforce Knowledge được đồng bộ thông qua **AWS AppFlow**, với các luồng riêng cho từng BU:

- `auto_kb`

- `credit_kb`

- `payments_kb`

AppFlow hỗ trợ:

- **On-Demand Flow**: Lấy toàn bộ dữ liệu KB

- **Incremental Flow**: Đồng bộ các cập nhật mới

Dữ liệu được lưu trữ trong **Amazon S3** theo prefix tương ứng.

---

## B. Phân tách dữ liệu trong Amazon S3

Mỗi BU được lưu tại prefix riêng:

- `auto_kb/`

- `credit_kb/`

- `payments_kb/`

S3 Event Notifications được cấu hình để kích hoạt bước xử lý tiếp theo.

---

## C. Gửi sự kiện S3 đến Amazon SQS

Khi có dữ liệu mới:

- S3 gửi sự kiện đến **Amazon SQS**

- SQS đảm bảo tách rời và xử lý ổn định cho pipeline downstream

---

## D. Xử lý dữ liệu bằng AWS Lambda

Lambda thực hiện:

- Làm sạch (sanitize) HTML

- Ghi tệp HTML vào các bucket:

  - `qic-auto`

  - `qic-credit`

  - `qic-payments`

Xử lý các trạng thái:

- **Publish/Update** → ghi đè nội dung

- **Archived** → xóa nội dung khỏi S3

---

## E. Đồng bộ KB với Amazon Q in Connect

Mỗi bucket S3 được liên kết với một KB EXTERNAL trong QiC:

- `qic-auto-kb`

- `qic-credit-kb`

- `qic-payments-kb`

QiC tự động đồng bộ khi S3 thay đổi.

---

## F. Tích hợp Amazon Connect & AI Agents

Luồng xử lý:

1\. Khách hàng gọi vào Amazon Connect  

2\. IVR xác định loại LOB  

3\. AWS Lambda cập nhật Agent Session  

4\. Gán KB tương ứng thông qua `updateSession()`

---

## Điều kiện tiên quyết

### AWS

- Tài khoản AWS hợp lệ

- AWS CLI & CDK CLI

- Quyền IAM cần thiết

### Amazon Connect

- Connect instance đang hoạt động

- Queue & quyền quản trị

### Salesforce

- Salesforce Org có API

- Salesforce Knowledge đã bật

- Connected App & OAuth scopes

---

## Triển khai CDK

```bash

cdk bootstrap aws://ACCOUNT/REGION

cdk diff

cdk deploy
```

Tài nguyên được tạo:

AppFlow Flows

Lambda Functions

IAM Roles

Amazon Q KBs

Amazon Connect integrations

Xác minh & Vận hành

Kiểm tra CloudFormation

Chạy AppFlow On-Demand & Scheduled

Kiểm tra dữ liệu tại S3

Monitor CloudWatch Logs

Dọn dẹp tài nguyên

bash

Copy code

cdk destroy

Kết luận

Giải pháp này cho phép các trung tâm liên hệ mở rộng quy mô hiệu quả bằng cách triển khai các tác tử AI theo từng LOB, giúp cung cấp nội dung chính xác, theo ngữ cảnh và theo thời gian thực cho nhân viên hỗ trợ, đồng thời nâng cao trải nghiệm khách hàng.
