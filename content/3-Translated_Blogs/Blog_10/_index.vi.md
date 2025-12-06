+++
title = "Blog 10"
weight = 10
chapter = false
pre = "<b>3.10.</b>"
+++

# Empower đã mở rộng quy mô đảm bảo chất lượng trung tâm liên hệ như thế nào với Amazon Connect và Amazon Bedrock

**Tác giả:** Marcos Ortiz, [Illan Geller (Accenture)](https://www.accenture.com/), [Ozlem Celik-Tinmaz (Accenture)](https://www.accenture.com/), [Prabhu Akula (Accenture)](https://www.accenture.com/), và [Ryan Baham (Empower)](https://www.empower.com/)  
**Ngày:** 04/08/2025  
**Thể loại:** Amazon Bedrock, Amazon Connect, Amazon Transcribe, Trung tâm liên hệ, AI tạo sinh

---

## Giới thiệu

[**Empower**](https://www.empower.com/) là một trong những công ty dịch vụ tài chính hàng đầu tại Hoa Kỳ, phục vụ hơn 18 triệu khách hàng với tổng tài sản quản lý lên đến 1,8 nghìn tỷ USD. Hàng năm, Empower xử lý khoảng 10 triệu cuộc gọi thông qua các trung tâm liên hệ của mình.

Để duy trì chất lượng dịch vụ ở quy mô lớn như vậy, Empower đã hợp tác với [**AWS (Amazon Web Services)**](https://aws.amazon.com/) và Accenture nhằm chuyển đổi quy trình đảm bảo chất lượng (QA) bằng cách ứng dụng **AI tạo sinh**. Thông qua việc triển khai một giải pháp tùy chỉnh dựa trên [**Amazon Connect**](https://aws.amazon.com/connect/) và [**Amazon Bedrock**](https://aws.amazon.com/bedrock/), Empower đã mở rộng phạm vi kiểm soát chất lượng cuộc gọi lên **gấp 20 lần**, đồng thời rút ngắn thời gian đánh giá từ vài ngày xuống chỉ còn vài phút.

Giải pháp này được đưa vào môi trường sản xuất chỉ trong **7 tháng**, cho thấy hiệu quả của việc kết hợp hạ tầng AWS, chuyên môn triển khai của Accenture và tầm nhìn đổi mới công nghệ của Empower.

---

## Thách thức: Đảm bảo chất lượng thủ công ở quy mô lớn

Empower sử dụng khung đánh giá **GEDAC** (Greeting, Engagement, Discovery, Action, Close) để đo lường hiệu suất của tổng đài viên. Trước đây, việc đánh giá được thực hiện thủ công bởi các chuyên viên QA thông qua việc nghe lại ghi âm cuộc gọi và chấm điểm theo các tiêu chí định sẵn.

Cách làm này tồn tại nhiều hạn chế:
- Chỉ có thể đánh giá một tỷ lệ rất nhỏ trong số 10 triệu cuộc gọi mỗi năm  
- Tính không nhất quán giữa các chuyên viên đánh giá  
- Thời gian phản hồi chậm, đôi khi kéo dài hàng tuần  
- Không thể mở rộng khi lưu lượng cuộc gọi ngày càng tăng  

> “Để thực sự cải thiện trải nghiệm khách hàng trên quy mô lớn, chúng tôi buộc phải tái định hình hoàn toàn cách tiếp cận đối với việc đảm bảo chất lượng,”  
> — **Joe Mieras**, VP Dịch vụ Thành viên tại Empower

---

## Tổng quan giải pháp

Empower đã xây dựng một hệ thống QA tự động dựa trên:
- **Amazon Connect Contact Lens**: Gỡ băng cuộc gọi, loại bỏ PII
- **Amazon Bedrock + Claude 3.5 Sonnet**: Đánh giá nội dung cuộc gọi theo khung GEDAC
- **AWS Lambda, SQS, EventBridge, Step Functions**: Điều phối và xử lý hàng loạt

Giải pháp xử lý **5.000 bản ghi mỗi ngày**, đánh giá nhất quán và ghi kết quả trực tiếp trở lại Amazon Connect.

<p align="center">
  <img src="https://d2908q01vomqb2.cloudfront.net/af3e133428b9e25c55bc59fe534248e6a0c0f17b/2025/07/30/connect-blog-12941-image-1.png" width="60%"/>
  <br><em>Sơ đồ kiến trúc tổng thể</em>
</p>

---

## Quy trình xử lý

1. **Gỡ băng cuộc gọi** bằng Amazon Connect Contact Lens  
2. **Lưu trữ bản ghi** vào Amazon S3  
3. **EventBridge** phát hiện dữ liệu mới  
4. **Amazon SQS** quản lý hàng đợi xử lý  
5. **AWS Lambda & Step Functions** điều phối quy trình GEDAC  
6. **Amazon Bedrock** đánh giá bằng Claude 3.5 Sonnet  
7. **Kết quả** được ghi trực tiếp vào Amazon Connect  

---

## Lợi ích của việc tận dụng dịch vụ AWS có sẵn

- Tự động loại bỏ PII – đảm bảo tuân thủ ngay từ đầu  
- Không cần xây dựng pipeline ETL tùy chỉnh  
- Giao diện hiển thị kết quả sẵn có trong Amazon Connect  
- Dễ mở rộng và vận hành ở quy mô lớn  

> “Chúng tôi không cần phải phát minh lại bánh xe,”  
> — **Joseph Mieras**, VP Trải nghiệm Khách hàng tại Empower

---

## Bảo mật và AI có trách nhiệm

Giải pháp đáp ứng đầy đủ các yêu cầu về:
- Mã hóa dữ liệu khi truyền và khi lưu trữ  
- Kiểm soát truy cập theo vai trò  
- Ghi log kiểm toán đầy đủ  
- Giám sát mô hình AI liên tục  

Empower cũng thành lập **Ủy ban Quản trị AI**, đảm bảo việc triển khai AI phù hợp với các tiêu chuẩn pháp lý, đạo đức và minh bạch.

---

## Kết quả đạt được

- Phạm vi QA tăng **20 lần**  
- Thời gian đánh giá giảm từ **vài ngày xuống vài phút**  
- Đánh giá nhất quán và có khả năng giải thích  
- Nhân sự QA tập trung vào các ca phức tạp và huấn luyện chuyên sâu  

---

## Bài học kinh nghiệm

- Bắt đầu từ mục tiêu kinh doanh rõ ràng  
- Đầu tư nghiêm túc vào thiết kế prompt  
- Lên kế hoạch mở rộng ngay từ đầu  
- AI **hỗ trợ**, không thay thế con người  
- Cải tiến liên tục dựa trên phản hồi thực tế  

---

## Hướng phát triển tiếp theo

Empower đang xây dựng một **nền tảng AI tạo sinh tập trung** cho hơn 1.500 lập trình viên, hỗ trợ:
- Quản trị AI tập trung  
- Theo dõi chi phí và hiệu suất  
- Tái sử dụng mô hình và best practices  

Giải pháp QA trung tâm liên hệ là nền tảng cho chiến lược AI dài hạn của Empower trong toàn doanh nghiệp.
