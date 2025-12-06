+++

title = "Blog 11"

weight = 11

chapter = false

pre = "<b> 3.11. </b>"

+++

# Làm thế nào để quản lý bot AI bằng AWS WAF và tăng cường bảo mật

**Tác giả:**  

Kartik Bheemisetty, David MacDonald

**Ngày:** 01/08/2025

**Danh mục:** AWS WAF, Best Practices, Security, Identity & Compliance, Technical Guide, Thought Leadership

---

## Giới thiệu

Trình thu thập dữ liệu web (*web crawler*) đầu tiên được tạo ra vào năm 1993 nhằm đo lường quy mô của World Wide Web. Kể từ đó, crawler đã phát triển thành những **bot hiện đại được hỗ trợ bởi AI**, có khả năng tự động hóa và tự hành.

Ngày nay, Internet ngày càng bị chi phối bởi các **bot AI** tự động tương tác với ứng dụng nhằm phục vụ nhiều tác vụ liên quan đến trí tuệ nhân tạo.

Bot AI có thể được phân thành ba nhóm chính:

- **Bot AI cào dữ liệu (AI scrapers):**  

  Thu thập dữ liệu một cách có hệ thống từ ứng dụng nhằm phục vụ huấn luyện mô hình AI.

- **Công cụ AI (AI tools):**  

  Hiển thị và sử dụng dữ liệu ứng dụng thông qua các kỹ thuật như *Function Calling*.

- **Tác tử AI (AI agents):**  

  Có khả năng tự chủ điều hướng và tương tác động với ứng dụng để thực hiện các tác vụ phức tạp.

Mặc dù một số bot AI mang lại giá trị (ví dụ: tự động hóa các tác vụ lặp lại), nhưng **bot độc hại** có thể gây ra nhiều rủi ro nghiêm trọng đối với ứng dụng web như tiêu tốn tài nguyên, suy giảm hiệu năng hoặc gián đoạn dịch vụ. Nếu không được kiểm soát, các vấn đề này sẽ ảnh hưởng trực tiếp đến bảo mật, niềm tin người dùng và uy tín thương hiệu.

Bài viết này trình bày các rủi ro do bot AI gây ra và cách **phát hiện, quản lý bot AI bằng AWS WAF**.

---

## Điều kiện tiên quyết

Bài viết tập trung vào **AWS WAF** như tuyến phòng thủ đầu tiên để quan sát và quản lý hoạt động của bot AI nhắm vào ứng dụng.

Nếu bạn chưa bật AWS WAF, hãy bắt đầu bằng việc:

- Sử dụng **AWS Shield Network Security Director** để xác định các tài nguyên chưa được bảo vệ.

- Thiết lập cấu hình bảo mật ban đầu thông qua tích hợp một cú nhấp chuột để tạo **Web ACL**.

Các tích hợp được hỗ trợ:

- **Amazon CloudFront:** Bật AWS WAF trực tiếp từ CloudFront.

- **Application Load Balancer (ALB):** Bật AWS WAF trực tiếp từ ALB.

---

## Các vấn đề do bot AI gây ra

Sự bùng nổ của **Large Language Models (LLMs)** và các **tác tử AI tự hành** đã khiến hành vi của bot ngày càng tinh vi, đặt ra nhiều thách thức:

### 1. Sử dụng dữ liệu độc quyền để huấn luyện mô hình

Việc thu thập dữ liệu trái phép có thể gây rủi ro về sở hữu trí tuệ và làm giảm giá trị nội dung độc quyền của doanh nghiệp.

### 2. Suy giảm hiệu năng và gia tăng chi phí

Bot AI tạo ra lượng lớn lưu lượng truy cập, dẫn đến:

- Tăng chi phí truyền dữ liệu (DTO)

- Tiêu thụ tài nguyên backend

- Nguy cơ gián đoạn dịch vụ

### 3. Hành vi tự động không mong muốn

Bot AI có thể cạnh tranh trực tiếp với người dùng hợp lệ trong các quy trình nhạy cảm về thời gian như:

- Săn vé

- Mua hàng số lượng giới hạn

Các kỹ thuật thường được sử dụng:

- Function calling và tìm kiếm AI

- Tự động hóa trình duyệt (Playwright)

- Tương tác thông qua môi trường VM

---

## Xác định quy mô hoạt động của bot AI

Để đánh giá mức độ ảnh hưởng của bot AI:

1\. Thêm **AWS WAF Bot Control -- Common Inspection**

2\. Đặt rule ở chế độ **Count**

Sau vài ngày thu thập dữ liệu, bạn có thể phân tích:

- Loại bot

- Hành vi truy cập

- Khối lượng lưu lượng

Thông tin này giúp xây dựng chiến lược kiểm soát phù hợp trước khi triển khai các biện pháp chặn.

---

## Quản lý bot AI

### Chặn sớm bằng robots.txt

#### Kịch bản 1: Chặn bot tuân thủ quy tắc

Tệp `robots.txt` cho phép kiểm soát các bot tuân thủ tiêu chuẩn. Các dự án như **ai.robots.txt** cung cấp danh sách bot AI phổ biến để chặn sớm.

Ví dụ:

```txt

User-agent: Amazonbot

Disallow: /private/

Allow: /public/

Kịch bản 2: Kiểm soát cách bot sử dụng dữ liệu

Amazonbot: sử dụng header X-Robots-Tag: noarchive

Applebot: khai báo User-agent: Applebot-Extended trong robots.txt

Googlebot: sử dụng User-agent: Google-Extended

Lưu ý: Không phải bot nào cũng tôn trọng robots.txt, vì vậy cần kết hợp AWS WAF.

Sử dụng AWS WAF để bảo vệ nâng cao

Kịch bản 3: Giảm tác động hiệu năng và chi phí

Bot Control -- Common Inspection: Chặn bot AI tự nhận diện (CategoryAI)

Rate limiting & L7 DDoS protection: Bảo vệ trước lưu lượng bất thường

AWS WAF Challenge: Buộc bot thực hiện các phép kiểm tra tốn chi phí

Honeypot: Dẫn bot vào các endpoint giả để phát hiện hành vi độc hại

Kịch bản 4: Quản lý bot AI tự hành

Bot Control (Common & Targeted Inspection)

CAPTCHA

Các biện pháp xác thực nâng cao (bao gồm sinh trắc học)

Kết luận

Bot AI mang lại nhiều thách thức về bảo mật, hiệu năng và quyền kiểm soát dữ liệu. Bằng cách kết hợp robots.txt, AWS WAF Bot Control, giới hạn tỷ lệ, CAPTCHA và các kỹ thuật nâng cao khác, doanh nghiệp có thể:

Ngăn chặn bot độc hại

Giảm chi phí vận hành

Vẫn cho phép các bot hợp lệ mang lại giá trị

Để tìm hiểu sâu hơn, hãy theo dõi AWS WAF Security Blog và các tài nguyên thuộc lĩnh vực Security, Identity & Compliance của AWS.