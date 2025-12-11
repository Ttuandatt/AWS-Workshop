+++

title = "Blog 3"

weight = 3

chapter = false

pre = "<b> 3.3. </b>"

+++

# Khắc phục sự cố môi trường Elastic Beanstalk bằng Amazon Q Developer CLI

**Tác giả:** Adarsh Suresh, Chandu Utlapalli  

**Ngày:** 29/07/2025

**Chủ đề:** Amazon Q Developer, AWS Elastic Beanstalk, Technical How-to

---

## Giới thiệu

Các nhà phát triển làm việc với AWS thường đánh giá cao [AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/) nhờ khả năng đơn giản hóa việc triển khai và vận hành các ứng dụng web mà không cần phải quản lý trực tiếp cơ sở hạ tầng nền tảng. Chỉ cần tải mã nguồn ứng dụng lên, Elastic Beanstalk sẽ tự động xử lý các tác vụ như cấp phát tài nguyên, cân bằng tải, tự động mở rộng và giám sát hệ thống, cho phép các nhóm tập trung hoàn toàn vào việc phát triển ứng dụng.

Với sự [ra mắt](https://aws.amazon.com/about-aws/whats-new/2025/03/amazon-q-developer-cli-agent-command-line/) phiên bản **CLI agent** được tăng cường của [Amazon Q Developer](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-installing.html), cách tiếp cận trong quy trình phát triển và vận hành phần mềm đã có nhiều thay đổi tích cực. Amazon Q CLI không chỉ hỗ trợ viết code mà còn mở rộng sang các tác vụ vận hành, giúp các nhóm DevOps xử lý sự cố nhanh hơn và hiệu quả hơn.

Bên cạnh phát triển phần mềm, các nhà phát triển và đội ngũ DevOps thường dành nhiều thời gian cho các công việc vận hành như triển khai, kiểm thử ứng dụng trên nhiều môi trường và đặc biệt là khắc phục các lỗi liên quan đến triển khai hoặc tình trạng sức khỏe của hệ thống. Các tính năng dựa trên **agent thông minh (agentic features)** của Amazon Q CLI có thể đơn giản hóa đáng kể các tác vụ này bằng cách hỗ trợ phân tích, chẩn đoán và đề xuất giải pháp một cách tự động.

Khi một môi trường Elastic Beanstalk hiển thị tình trạng sức khỏe suy giảm hoặc gặp lỗi triển khai, Amazon Q CLI có thể đóng vai trò như một trợ lý kỹ thuật. Thay vì phải truy cập nhiều trang trên AWS Console và phân tích logs thủ công, nhà phát triển chỉ cần mô tả vấn đề thông qua một cuộc trò chuyện. Q CLI có thể giúp:

- Phân tích logs của instance

- Kiểm tra cấu hình môi trường

- Xác định các cấu hình sai trong ứng dụng

- Trích xuất thông báo lỗi liên quan

- Đề xuất các bước khắc phục phù hợp dựa trên các mẫu lỗi phổ biến

Ngoài ra, khi xử lý các vấn đề về tình trạng sức khỏe, Q CLI có thể kiểm tra trạng thái môi trường, mức sử dụng tài nguyên và các sự kiện gần đây để xác định các nguyên nhân như:

- Thiếu bộ nhớ (out-of-memory)

- Vấn đề kết nối

- Lỗi dependency

- Lỗi lặp lại trong application logs

Một trong những điểm mạnh nổi bật của Q CLI là khả năng kết nối thông tin giữa nhiều dịch vụ AWS. Ví dụ, nếu nguyên nhân sự cố đến từ cấu hình Amazon VPC hoặc quyền truy cập Amazon S3, Q CLI có thể phát hiện và đưa ra giải pháp tổng thể thay vì chỉ khắc phục từng phần riêng lẻ.

Nhờ đó, các tác vụ điều tra vốn có thể mất hàng giờ nay có thể được hoàn thành chỉ trong vài phút, giúp duy trì môi trường Elastic Beanstalk ổn định và giảm đáng kể thời gian downtime.

---

## Hướng dẫn giải pháp

### Điều kiện tiên quyết

Để có thể thực hiện các bước minh họa trong bài viết, bạn cần đáp ứng các yêu cầu sau:

1\. Một tài khoản AWS có quyền truy cập Elastic Beanstalk  

2\. Hiểu biết cơ bản về các [khái niệm Elastic Beanstalk](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/concepts.html) (applications, environments, deployments)  

3\. AWS CLI đã được [cài đặt](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) và cấu hình với các [quyền phù hợp](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/AWSHowTo.iam.managed-policies.html)  

4\. Amazon Q Developer CLI đã được [cài đặt và thiết lập](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-installing.html)  

5\. EB CLI đã được [cài đặt](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html) (không bắt buộc)  

6\. Ít nhất một môi trường Elastic Beanstalk web server đã được [tạo sẵn](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.environments.html) để thực hành khắc phục sự cố

> **Lưu ý:**  

> Các kịch bản trong bài viết được kiểm tra với Amazon Q Developer CLI sử dụng **Pro tier**, do tier này cung cấp giới hạn yêu cầu cao hơn. Tuy nhiên, Pro tier không bắt buộc để hiểu và áp dụng các ví dụ được trình bày.

---

## Khắc phục sự cố tình trạng môi trường

Xét một môi trường Elastic Beanstalk chạy **Node.js 22** trên **Amazon Linux 2023**, trong đó một phiên bản ứng dụng mới vừa được triển khai. Sau khi triển khai, môi trường hiển thị tình trạng sức khỏe ở mức **Warning**, với thông báo sự kiện sau:

```text

100% of requests failing with HTTP 5xx errors

![](/AWS-Workshop/images/img1-blog3.png)


Thông báo này có thể xuất phát từ nhiều nguyên nhân khác nhau, bao gồm:

Lỗi trong ứng dụng Node.js

Cấu hình reverse proxy không chính xác

Sự cố sử dụng tài nguyên

Các vấn đề khác liên quan đến hạ tầng hoặc dependency

Để điều tra thêm, chúng ta có thể sử dụng Amazon Q CLI. Bắt đầu một cuộc trò chuyện mới bằng cách chạy:

```bash
q chat
```

Sau đó đặt câu hỏi:

```
Why is my beanstalk environment nodejs-app in us-east-1 unhealthy?
```

Check the logs if required, and recommend steps to resolve the issue.

Amazon Q CLI sẽ tự động thu thập thông tin liên quan từ logs, cấu hình môi trường và các sự kiện gần đây, sau đó đưa ra phân tích cũng như đề xuất các bước khắc phục cụ thể.

{{< figurecaption src="/images/gif1-blog3" caption="Q CLI phân tích logs và đề xuất hướng khắc phục sự cố môi trường Elastic Beanstalk" >}}
