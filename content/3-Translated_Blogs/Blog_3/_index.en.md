+++
title = "Blog 3"
weight = 3
chapter = false
pre = " <b> 3.3. </b>"
+++

# **Khắc phục sự cố Môi trường Elastic Beanstalk bằng Amazon Q Developer CLI**

*Tác giả: Adarsh Suresh, Chandu Utlapalli – 29/7/2025*  
*Chủ đề: Amazon Q Developer, AWS Elastic Beanstalk, Technical How-to*

---

### Giới thiệu

Các nhà phát triển làm việc với AWS nhận thấy [AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/) là một dịch vụ vô giá giúp việc triển khai và chạy các ứng dụng web trở nên đơn giản mà không cần phải lo lắng về cơ sở hạ tầng nền tảng (underlying infrastructure). Bạn chỉ cần tải mã ứng dụng của mình lên, và Elastic Beanstalk sẽ tự động xử lý các chi tiết về cấp phát dung lượng (capacity provisioning), cân bằng tải (load balancing), điều chỉnh quy mô (scaling), và giám sát (monitoring), cho phép bạn tập trung vào việc viết code.

Với việc [phát hành](https://aws.amazon.com/about-aws/whats-new/2025/03/amazon-q-developer-cli-agent-command-line/) [CLI agent](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-installing.html) mới được tăng cường của Amazon Q Developer, chúng ta đã thấy cách Q CLI có thể được sử dụng để chuyển đổi [phương pháp tiếp cận quy trình phát triển phần mềm](https://aws.amazon.com/blogs/devops/introducing-the-enhanced-command-line-interface-in-amazon-q-developer/). 

Ngoài phát triển phần mềm, các nhà phát triển và đội ngũ DevOps có thể dành phần lớn thời gian của họ cho các tác vụ vận hành (operational tasks) như triển khai và kiểm thử mã của họ trên nhiều môi trường, bao gồm cả việc khắc phục sự cố các lỗi liên quan đến triển khai (deployment related failures) hoặc các vấn đề về tình trạng ứng dụng (application health issues). Các tính năng dựa trên tác nhân (agentic features) mới của Q CLI có thể được sử dụng để đơn giản hóa đáng kể quy trình này bằng cách giúp bạn xác định và giải quyết các vấn đề vận hành theo cách hiệu quả hơn.

Khi khắc phục sự cố môi trường Elastic Beanstalk, Q CLI trở thành người bạn đồng hành không thể thiếu. Khi các môi trường hiển thị tình trạng sức khỏe bị suy giảm (degraded health) hoặc lỗi triển khai (deployment failures), các nhà phát triển có thể sử dụng Q CLI để nhanh chóng điều tra mà không cần phải điều hướng qua nhiều trang AWS console hoặc phân tích nhiều logs thủ công. 

Ví dụ, khi đối mặt với lỗi triển khai, bạn có thể chạy q chat để bắt đầu một cuộc trò chuyện mới và mô tả vấn đề. Q CLI có thể giúp phân tích instance logs, kiểm tra cấu hình môi trường (environment configurations), và xác định các cấu hình sai (misconfigurations) trong ứng dụng. Nó có thể lấy các thông báo lỗi liên quan từ Elastic Beanstalk logs và đề xuất các biện pháp khắc phục cụ thể dựa trên các mẫu lỗi mà nó nhận ra.

Khi giải quyết các vấn đề về tình trạng sức khỏe (health issues), các nhà phát triển có thể yêu cầu Q CLI kiểm tra trạng thái môi trường, mức sử dụng tài nguyên (resource utilization) và các sự kiện gần đây. Nó có thể xác định xem một ứng dụng có đang gặp vấn đề thiếu bộ nhớ (out of memory problems), vấn đề kết nối (connectivity issues), hay lỗi liên quan đến dependency hay không. Q CLI cũng có thể kiểm tra application logs để tìm các lỗi lặp lại có thể gây ra suy giảm tình trạng sức khỏe (health degradation). 

Điều mà các nhà phát triển đánh giá cao nhất là cách Q CLI kết nối các điểm giữa các dịch vụ AWS khác nhau. Nếu một môi trường Elastic Beanstalk gặp sự cố do vấn đề cấu hình Amazon VPC cơ bản hoặc vấn đề quyền Amazon S3, Q CLI có thể xác định các kết nối này và cung cấp các giải pháp toàn diện (holistic solutions).

Việc tiết kiệm thời gian là rất đáng kể – những gì trước đây mất hàng giờ điều tra trên nhiều trang AWS console giờ đây chỉ mất vài phút với các truy vấn Q CLI có mục tiêu. Điều này đã cải thiện đáng kể khả năng của các nhà phát triển trong việc duy trì các môi trường khỏe mạnh và nhanh chóng giải quyết các vấn đề khi chúng phát sinh. 

Dưới đây, chúng tôi sẽ hướng dẫn bạn một số ví dụ về cách bạn có thể sử dụng Q CLI để khắc phục một số sự cố mà bạn có thể gặp phải khi quản lý môi trường Elastic Beanstalk.


### Hướng dẫn Giải pháp

**Điều kiện Tiên quyết**

Nếu bạn muốn làm theo trên máy tính của riêng mình, vui lòng đảm bảo bạn hoàn thành các điều kiện tiên quyết sau:

1. Một tài khoản AWS có quyền truy cập Elastic Beanstalk
2. Sự quen thuộc cơ bản với các [khái niệm](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/concepts.html) Elastic Beanstalk (environments, applications, deployments)
3. AWS CLI [được cài đặt](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) và cấu hình với các [quyền](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/AWSHowTo.iam.managed-policies.html) thích hợp để truy cập tài nguyên Elastic Beanstalk, và thu thập logs
4. AWS Q Developer CLI [được cài đặt](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-installing.html) và thiết lập
5. EB CLI [được cài đặt](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html) và thiết lập (tùy chọn)
6. Các môi trường web server Elastic Beanstalk đã [được tạo](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.environments.html) để khắc phục sự cố
Bây giờ chúng ta hãy đi sâu vào việc khắc phục các sự cố Elastic Beanstalk cụ thể với Q CLI. Tất cả các kịch bản dưới đây đều được kiểm tra với Amazon Q Developer CLI bằng gói đăng ký [Pro tier](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/q-pro-tier.html) vì nó cung cấp giới hạn yêu cầu cao hơn (higher request limits), nhưng điều này không bắt buộc cho mục đích của bản demo này.

**Khắc phục sự cố tình trạng môi trường**

Hãy xem xét một môi trường Elastic Beanstalk đang chạy Node.js 22 trên Amazon Linux 2023, nơi chúng ta sẽ triển khai một phiên bản ứng dụng mới. Sau khi triển khai một phiên bản ứng dụng mới vào môi trường Elastic Beanstalk dựa trên Node.js của chúng ta, chúng tôi nhận thấy rằng tình trạng sức khỏe (health status) của nó đã chuyển sang trạng thái “Warning” (Cảnh báo) với thông báo sau hiển thị trong các sự kiện môi trường (environment events):

```
100% of requests failing with HTTP 5xx errors
```

![](/AWS-Workshop/images/img1-blog3.png)



Thông báo sự kiện này có thể là kết quả của một số vấn đề, bao gồm nhưng không giới hạn ở lỗi ứng dụng Node.js, sự cố cấu hình reverse proxy, vấn đề sử dụng tài nguyên (resource utilization issues) v.v.. 

Hãy sử dụng Q CLI để giúp chúng ta điều tra thêm. Chúng ta sẽ bắt đầu một cuộc trò chuyện mới với agent bằng cách chạy ```q chat```, và hỏi câu hỏi sau:

```Why is my beanstalk environment nodejs-app in us-east-1 unhealthy? Check the logs if required, and recommend steps to resolve the issue``` (Tại sao môi trường beanstalk nodejs-app của tôi ở us-east-1 không khỏe mạnh? Kiểm tra các logs nếu cần, và đề xuất các bước để giải quyết vấn đề)

{{< figurecaption src="/images/gif1-blog3" caption="" >}}
