+++
title = "Blog 10"
weight =  10
chapter = false
pre = " <b> 3.10. </b>"
+++

# Empower đã mở rộng quy mô đảm bảo chất lượng trung tâm liên hệ như thế nào với Amazon Connect và Amazon Bedrock

**Tác giả:** Marcos Ortiz, [Illan Geller (Accenture)](https://www.accenture.com/), [Ozlem Celik-Tinmaz (Accenture)](https://www.accenture.com/), [Prabhu Akula (Accenture)](https://www.accenture.com/), và [Ryan Baham (Empower)](https://www.empower.com/)  

**Ngày:** 04 tháng 8 năm 2025  

**Thể loại:** Amazon Bedrock, Amazon Connect, Amazon Transcribe, Trung tâm liên hệ, AI Tạo sinh  

---

### Giới thiệu

[**Empower**](https://www.empower.com/) là một công ty dịch vụ tài chính hàng đầu phục vụ hơn 18 triệu người Mỹ với 1,8 nghìn tỷ đô la tài sản đang quản lý. Họ tiếp nhận khoảng 10 triệu cuộc gọi của khách hàng hàng năm thông qua các trung tâm chăm sóc của mình. Để duy trì dịch vụ xuất sắc ở quy mô này, Empower đã hợp tác với [**AWS (Amazon Web Services)**](https://aws.amazon.com/) và Accenture để chuyển đổi quy trình đảm bảo chất lượng (QA) bằng cách sử dụng AI tạo sinh. Bằng cách triển khai một giải pháp tùy chỉnh với [**Amazon Connect**](https://aws.amazon.com/connect/) và [**Amazon Bedrock**](https://aws.amazon.com/bedrock/), Empower có thể mở rộng phạm vi bao phủ cuộc gọi để đảm bảo chất lượng lên 20 lần, hiện phân tích hàng nghìn bản gỡ băng cuộc gọi hàng ngày và giảm thời gian xem xét QA từ vài ngày xuống còn vài phút.

Trong bài viết này, chúng tôi khám phá cách thức sự hợp tác ba bên này đã mang lại một giải pháp AI tạo sinh sẵn sàng cho sản xuất từ [giai đoạn] thử nghiệm đến [giai đoạn] sản xuất chỉ trong 7 tháng. Điều này chứng minh sức mạnh của việc kết hợp công nghệ AWS, chuyên môn triển khai của Accenture, và tầm nhìn của Phòng thí nghiệm Đổi mới Công nghệ của Empower.

### Thách thức: Đảm bảo chất lượng thủ công trên quy mô lớn

Trung tâm liên hệ của Empower sử dụng một khuôn khổ đánh giá toàn diện gọi là GEDAC (Chào hỏi, Tương tác, Khám phá, Hành động, Kết thúc) để đánh giá hiệu suất của nhân viên dựa trên năm lĩnh vực kỹ năng chính. Mỗi lĩnh vực bao gồm nhiều kỹ năng phụ, từ chào hỏi khách hàng một cách thích hợp đến duy trì thái độ thân thiện và phản hồi kịp thời. Các chuyên viên phân tích chất lượng đã xem xét thủ công các bản ghi cuộc gọi và chấm điểm nhân viên dựa trên các tiêu chí định trước cho mỗi lĩnh vực kỹ năng. Quy trình thủ công này đặt ra một số thách thức. Với việc những người đánh giá (con người) chỉ có thể đánh giá một tập hợp con nhỏ trong số 10 triệu cuộc gọi hàng năm, phạm vi bao phủ vẫn còn hạn chế. Quy trình đánh giá thủ công cũng tạo cơ hội cho các đánh giá không nhất quán, vì những người đánh giá khác nhau có thể chấm điểm các tương tác giống hệt nhau một cách khác nhau. Hơn nữa, tính chất tốn thời gian của mỗi lần đánh giá đã hạn chế số lượng các bài xem xét có thể thực hiện. Điều này dẫn đến phản hồi chậm trễ, với việc các nhân viên nhận được đánh giá hiệu suất sau nhiều ngày hoặc thậm chí nhiều tuần kể từ khi tương tác với khách hàng. Khi khối lượng cuộc gọi tiếp tục tăng, những hạn chế về khả năng mở rộng này khiến cho việc bao phủ toàn diện ngày càng khó đạt được.

“Chúng tôi nhận ra rằng để thực sự nâng cao trải nghiệm khách hàng trên quy mô lớn, chúng tôi cần phải tái định hình cơ bản phương pháp tiếp cận của mình đối với việc đảm bảo chất lượng,” Joe Mieras, Phó Giám đốc (VP) Dịch vụ Thành viên tại Empower, cho biết. “Quy trình thủ công đơn giản là không thể theo kịp tốc độ tăng trưởng và cam kết của chúng tôi về dịch vụ xuất sắc.”

Thách thức này hoàn toàn phù hợp với sứ mệnh của Phòng thí nghiệm Đổi mới Công nghệ của Empower là khơi dậy sự đổi mới bằng cách thử nghiệm các khả năng mới, tăng tốc các công nghệ mới và tạo sự khác biệt cho trải nghiệm khách hàng thông qua việc khám phá an toàn và minh bạch. Phòng thí nghiệm, vốn đã thu hút hơn 11.000 cộng sự thông qua các bản demo và roadshow trong khi nghiên cứu hơn 80 công nghệ mới nổi, đã xác định QA cuộc gọi là một ứng cử viên hàng đầu cho việc thử nghiệm AI tạo sinh.

### Tổng quan giải pháp

Làm việc với AWS và Accenture, Empower đã phát triển một giải pháp QA tự động tận dụng Amazon Connect Contact Lens, dịch vụ mà họ đã kích hoạt và đang cung cấp các bản gỡ băng chất lượng cao, đã loại bỏ PII. Bằng cách kết hợp các bản gỡ băng sẵn sàng sử dụng này với Amazon Bedrock và Claude 3.5 Sonnet của Anthropic để đánh giá thông minh, đội ngũ đã tránh được hàng tuần phát triển ETL (trích xuất, chuyển đổi và tải) và che giấu dữ liệu. Giải pháp xử lý 5.000 bản gỡ băng đã được loại bỏ (thông tin nhạy cảm) mỗi ngày theo lô, đánh giá nhân viên trên tất cả các hạng mục GEDAC.

Sơ đồ sau đây minh họa kiến trúc giải pháp cấp cao:

<p align="center">
  <img src="https://d2908q01vomqb2.cloudfront.net/af3e133428b9e25c55bc59fe534248e6a0c0f17b/2025/07/30/connect-blog-12941-image-1.png" 
       alt="Sơ đồ kiến trúc cấp cao" 
       width="50%">
  <br>
  <em>Sơ đồ kiến trúc cấp cao</em>
</p>

### 🧩 Quy trình làm việc bao gồm các bước sau:

1.  **Gỡ băng cuộc gọi** [**Amazon Connect Contact Lens**](https://aws.amazon.com/connect/contact-lens/) tự động gỡ băng các cuộc gọi của khách hàng với độ chính xác cao, loại bỏ PII và ghi lại sự phân tách người nói, cảm xúc, cùng các siêu dữ liệu khác.  
    Sau đó, nó lưu trữ các tệp gỡ băng vào [**Amazon S3**](https://aws.amazon.com/s3/), loại bỏ nhu cầu về các đường ống dữ liệu tùy chỉnh.

2.  **Thông báo sự kiện** [**Amazon EventBridge**](https://aws.amazon.com/eventbridge/) phát hiện các tệp gỡ băng mới trong S3 và tự động kích hoạt các hành động tiếp theo.

3.  **Quản lý hàng đợi** EventBridge gửi một tin nhắn đến [**Amazon SQS**](https://aws.amazon.com/sqs/), dịch vụ này sẽ quản lý hàng đợi các bản gỡ băng cần được xử lý, đảm bảo việc xử lý hàng loạt đáng tin cậy và có khả năng mở rộng.

4.  **Xử lý hàng loạt** Các hàm [**AWS Lambda**](https://aws.amazon.com/lambda/) thăm dò hàng đợi SQS và truy xuất các lô gỡ băng để xử lý.

5.  **Điều phối quy trình GEDAC** Hàm Lambda kích hoạt [**AWS Step Functions**](https://aws.amazon.com/step-functions/) liên kết để đánh giá mỗi bản gỡ băng cuộc gọi dựa trên tất cả các chủ đề được định nghĩa trong khuôn khổ GEDAC.

6.  **Đánh giá bằng AI** Step Functions gửi các bản gỡ băng đến [**Amazon Bedrock**](https://aws.amazon.com/bedrock/), nơi **Claude 3.5 Sonnet** đánh giá hiệu suất của nhân viên dựa trên năm hạng mục GEDAC.

7.  **Đưa kết quả vào hàng đợi** Kết quả đánh giá được gửi đến một hàng đợi SQS khác để xử lý và gửi đi một cách có kiểm soát.

8.  **Xử lý kết quả** Một hàm Lambda thứ hai xử lý các kết quả đánh giá từ hàng đợi.

9.  **Gửi (Phân phối) kết quả** Hàm Lambda ghi kết quả đánh giá trở lại [**Amazon Connect**](https://aws.amazon.com/connect/) bằng cách sử dụng **API Đánh giá Nhân viên (Agent Evaluation API)**.  
    Các nhà quản lý sau đó có thể xem kết quả trực tiếp trong **giao diện Quản lý Chất lượng Amazon Connect** hiện có — **không yêu cầu phát triển GUI tùy chỉnh**.

### Tận dụng các dịch vụ AWS

Một yếu tố then chốt trong việc phát triển và triển khai nhanh chóng của giải pháp là việc sử dụng các dịch vụ và tính năng AWS hiện có, thay vì xây dựng mọi thứ từ đầu.

Khả năng kết hợp Amazon Connect cho trung tâm liên hệ đám mây của chúng tôi với các dịch vụ AWS khác, thay vì xây dựng từ đầu, là một yếu tố then chốt trong việc phát triển và triển khai nhanh chóng của giải pháp.

Amazon Connect Contact Lens đã cung cấp sẵn [tính năng] loại bỏ tự động thông tin nhận dạng cá nhân (PII) trong các bản gỡ băng cuộc gọi. Điều này loại bỏ nhu cầu Empower phải triển khai các đường ống ETL tùy chỉnh và các giải pháp che giấu dữ liệu, giúp giảm đáng kể thời gian phát triển và đảm bảo tuân thủ các yêu cầu bảo vệ dữ liệu ngay từ ngày đầu tiên. Đội ngũ [phát triển] có thể tập trung vào việc xây dựng logic đánh giá khi mà các vấn đề về quyền riêng tư dữ liệu và tuân thủ đã được giải quyết sẵn.

Contact Lens tự động truyền các tệp gỡ băng cuộc gọi đến Amazon S3, kích hoạt quy trình xử lý tiếp theo. Sự tích hợp gốc này đã loại bỏ nhu cầu về các giải pháp di chuyển dữ liệu tùy chỉnh và cung cấp một nền tảng đáng tin cậy, có khả năng mở rộng cho đường ống xử lý hàng loạt.

Ngoài ra, các tính năng đánh giá hiệu suất nhân viên hiện có sẵn trong Amazon Connect đã cung cấp một giao diện người dùng có sẵn để hiển thị kết quả đánh giá. Giải pháp sử dụng API Đánh giá Nhân viên của Amazon Connect để ghi các kết quả đánh giá từ Bedrock trực tiếp vào Amazon Connect, nơi các nhà quản lý có thể xem chúng cùng với các chỉ số chất lượng khác trong một giao diện quen thuộc. “Chúng tôi không cần phải ‘phát minh lại bánh xe’,” Joseph Mieras, Phó Giám đốc (VP) Trải nghiệm Khách hàng tại Empower, giải thích. “Bằng cách sử dụng API Quản lý Chất lượng của Amazon Connect, chúng tôi có thể trình bày các đánh giá do AI tạo ra trên chính giao diện mà đội ngũ của chúng tôi vốn đã sử dụng, giúp cải thiện đáng kể việc tiếp nhận và giảm thiểu các yêu cầu đào tạo.”

<p align="center">
  <img src="https://d2908q01vomqb2.cloudfront.net/af3e133428b9e25c55bc59fe534248e6a0c0f17b/2025/07/30/connect-blog-12941-image-2.png" 
       alt="Giao diện Người dùng Đồ họa (GUI) Đánh giá Nhân viên của Amazon Connect" 
       width="50%">
  <br>
  <em>Giao diện Người dùng Đồ họa (GUI) Đánh giá Nhân viên của Amazon Connect</em>
</p>

### Tại sao chọn triển khai tùy chỉnh thay vì các tính năng có sẵn
Các tính năng đánh giá hiệu suất nhân viên của Amazon Connect cung cấp chức năng có sẵn tuyệt vời cho nhiều tổ chức. Tuy nhiên, khuôn khổ GEDAC của Empower là kết quả của nhiều thập kỷ tinh chỉnh dành riêng cho hoạt động kinh doanh của họ. Sự linh hoạt của Amazon Connect trong việc tùy chỉnh giải pháp của họ bằng cách sử dụng Amazon Bedrock đã cho phép Empower triển khai các tiêu chí đánh giá chính xác của họ, đồng thời duy trì khả năng phát triển giải pháp khi nhu cầu của họ thay đổi.

Giải pháp này cho phép Empower triển khai phương pháp luận GEDAC của họ với sự đánh giá có sắc thái trên nhiều kỹ năng phụ, nắm bắt được các tiêu chí cụ thể làm nên sự độc đáo cho khuôn khổ của họ. Nó cung cấp sự linh hoạt để điều chỉnh các câu lệnh và tiêu chí đánh giá mà không cần thay đổi hệ thống, cho phép việc tinh chỉnh liên tục dựa trên nhu cầu kinh doanh. Ngoài ra, nó còn cung cấp lý giải chi tiết cho mỗi điểm số, mang lại khả năng giải thích cần thiết để hỗ trợ việc huấn luyện nhân viên hiệu quả và cải thiện hiệu suất.

Các cân nhắc về Bảo mật và AI có trách nhiệm
Giải pháp áp dụng các biện pháp bảo mật mạnh mẽ để bảo vệ thông tin nhạy cảm trong suốt quá trình đánh giá. Tất cả dữ liệu đều được mã hóa cả khi đang truyền và khi lưu trữ, để bảo vệ khỏi sự truy cập trái phép. Các biện pháp kiểm soát truy cập dựa trên vai trò đảm bảo rằng chỉ những nhân sự được ủy quyền mới có thể xem kết quả đánh giá, duy trì việc quản trị dữ liệu nghiêm ngặt. Các chính sách tự động quản lý vòng đời dữ liệu theo các yêu cầu quy định, trong khi đó, việc ghi nhật ký kiểm toán toàn diện cung cấp một dấu vết hoàn chỉnh về mọi hoạt động của hệ thống cho mục đích tuân thủ và giám sát an ninh.

Empower đã triển khai một khuôn khổ quản trị AI mạnh mẽ nhằm giải quyết nhiều khía cạnh của AI có trách nhiệm. Một Ủy ban Quản trị AI cung cấp sự giám sát tập trung đối với mọi hoạt động sử dụng và phát triển AI, xem xét và đánh giá rủi ro trước khi thực thi. Công ty đã thiết lập các rào cản pháp lý và tuân thủ toàn diện cho việc phát triển và sử dụng mô hình AI, cùng với một quy trình giám sát mô hình nhằm duy trì một kho AI tập trung với quyền sở hữu minh bạch, sự giám sát liên tục, và chứng nhận hàng năm được chính thức hóa.

<p align="center">
  <img src="https://d2908q01vomqb2.cloudfront.net/af3e133428b9e25c55bc59fe534248e6a0c0f17b/2025/07/30/connect-blog-12941-image-3.png" 
       alt="Phương pháp tiếp cận AI có trách nhiệm của Empower" 
       width="50%">
  <br>
  <em>Phương pháp tiếp cận AI có trách nhiệm của Empower</em>
</p>


Phương pháp tiếp cận đa diện này chủ động hoạt động để tránh thiên vị AI, ngăn chặn gian lận, bảo vệ khỏi lộ lọt dữ liệu, quản lý rủi ro pháp lý và quy định, và đảm bảo chất lượng dữ liệu huấn luyện. Khuôn khổ này cũng nhấn mạnh tính minh bạch, cho phép các nhân viên xem xét các đánh giá của AI và hiểu lý giải điểm số, đồng thời duy trì sự giám sát của con người, nơi các chuyên viên phân tích chất lượng có thể xem xét và ghi đè các đánh giá của AI. Việc giám sát liên tục đảm bảo sự đánh giá không ngừng về hiệu suất và tính công bằng của mô hình, tạo ra một sự triển khai AI bền vững và có đạo đức.

### Sức mạnh của quan hệ đối tác AWS, Accenture và Empower
Việc triển khai thành công giải pháp này nhấn mạnh giá trị của quan hệ đối tác chiến lược trong việc cung cấp các giải pháp AI doanh nghiệp. Amazon Connect, với tư cách là giải pháp trung tâm liên hệ của Empower, cung cấp các bản gỡ băng chất lượng cao thiết yếu cho việc đánh giá chính xác. Việc tích hợp Amazon Bedrock cung cấp quyền truy cập vào các mô hình nền tảng tiên tiến với bảo mật cấp doanh nghiệp. Đội ngũ đã đảm bảo thiết kế tối ưu cho quy mô và hiệu suất thông qua hướng dẫn về kiến trúc, và chia sẻ các bài học kinh nghiệm từ các lần triển khai tương tự trong ngành dịch vụ tài chính, mang lại các thông lệ tốt nhất có giá trị cho dự án.

Vai trò của Accenture với tư cách là đối tác triển khai là rất quan trọng trong việc chuyển hóa các yêu cầu của Empower thành một giải pháp sẵn sàng cho sản xuất. Đội ngũ đã phát triển các câu lệnh phức tạp nắm bắt chính xác các tiêu chí đánh giá GEDAC, tiến hành nhiều vòng kiểm thử và tối ưu hóa với các chuyên gia trong lĩnh vực để đảm bảo độ chính xác. Họ đảm bảo tích hợp liền mạch với hệ thống công nghệ hiện có của Empower, đồng thời hỗ trợ quá trình chuyển đổi của tổ chức sang việc đảm bảo chất lượng được tăng cường bằng AI thông qua quản lý thay đổi toàn diện. Đội ngũ đã làm việc chặt chẽ với các chuyên viên phân tích chất lượng của Empower để hiểu rõ các sắc thái trong tiêu chí đánh giá của họ, sau đó chuyển hóa chuyên môn đó thành các câu lệnh mà Claude 3.5 Sonnet có thể thực thi một cách nhất quán và chính xác. Quá trình hợp tác này đảm bảo mô hình AI có thể tái tạo độ sâu và tính đặc thù của các đánh giá của con người, trong khi vẫn duy trì sự nhất quán cần thiết cho các hoạt động ở quy mô lớn.

Sự đóng góp của Empower vượt xa vai trò là một khách hàng—họ là một đối tác tích cực trong thiết kế giải pháp. Họ cung cấp sự hiểu biết sâu sắc về hoạt động của trung tâm liên hệ và các yêu cầu về chất lượng, chia sẻ phương pháp luận GEDAC chi tiết và các tiêu chí chấm điểm, vốn là nền tảng của hệ thống đánh giá AI. Các chuyên viên phân tích chất lượng của họ đã đóng một vai trò quan trọng trong việc xác thực các đánh giá của AI và cung cấp phản hồi để cải tiến liên tục. Thêm vào đó, Empower định vị sáng kiến này trong chiến lược chuyển đổi AI tạo sinh rộng lớn hơn của họ, đảm bảo sự phù hợp với các mục tiêu dài hạn của tổ chức và tạo ra một kế hoạch chi tiết cho các lần triển khai AI trong tương lai trên toàn doanh nghiệp.

### Kết quả và tác động kinh doanh
Với giải pháp này, Empower đã chứng kiến sự gia tăng đáng kể gấp 20 lần về phạm vi bao phủ cuộc gọi QA, mở rộng từ việc chỉ xem xét một mẫu con sang khả năng chấm điểm tất cả các cuộc gọi. Thời gian đánh giá giảm từ vài ngày xuống còn vài phút, trong khi các đánh giá AI được chuẩn hóa đã loại bỏ sự thiếu nhất quán giữa những người đánh giá vốn trước đây là một thách thức về tính nhất quán. Amazon Connect xử lý các biến động về khối lượng hàng ngày mà không yêu cầu thêm tài nguyên, cung cấp khả năng mở rộng cần thiết cho các hoạt động đang phát triển của Empower. Thêm vào đó, các nhóm QA có thể ưu tiên các cuộc gọi cần nhiều phản hồi nhất, đảm bảo những người đánh giá (con người) tập trung nỗ lực vào nơi họ tạo ra nhiều giá trị nhất.

Việc triển khai đã thúc đẩy tối ưu hóa chi phí đáng kể bằng cách tự động hóa các tác vụ thủ công lặp đi lặp lại, cho phép các chuyên viên phân tích QA chuyển hướng chuyên môn của họ sang các hoạt động có giá trị cao như xử lý các trường hợp phức tạp, phát triển các chương trình đào tạo và cung cấp huấn luyện nhân viên được cá nhân hóa. Thay vì dành hàng giờ cho các đánh giá thông thường, nhân sự QA giờ đây tập trung vào các cải tiến chiến lược giúp nâng cao trực tiếp trải nghiệm của khách hàng. Ngoài lợi ích về hiệu suất tức thì, giải pháp còn tạo ra một nền tảng có thể mở rộng, có thể tái sử dụng cho các trường hợp sử dụng khác trong toàn tổ chức.

Các nhân viên nhận được thông tin chi tiết về hiệu suất trong vòng vài giờ thay vì vài tuần, làm thay đổi chu trình phản hồi. Các giải thích chi tiết đi kèm với mỗi đánh giá sẽ giúp các nhà quản lý cung cấp huấn luyện có mục tiêu dựa trên các ví dụ tương tác cụ thể. Khả năng phân tích cũng có thể tiết lộ các mẫu hình giữa các nhóm và loại cuộc gọi mà trước đây không thể thấy rõ, cho phép cải tiến dựa trên dữ liệu đối với quy trình và đào tạo. Việc cập nhật mô hình thường xuyên mang đến các thông lệ tốt nhất mới và một chu trình cải tiến liên tục.

“Tác động của những giải pháp như thế này sẽ mang tính chuyển đổi,” Kyle Caffey, Phó Giám đốc (VP) của Innovation Lab tại Empower, cho biết. “Chúng ta không chỉ cải thiện đáng kể hiệu suất hoạt động của mình mà còn thúc đẩy chất lượng tốt hơn, điều này trực tiếp chuyển thành trải nghiệm khách hàng được cải thiện.”

### Bài học kinh nghiệm và các thông lệ tốt nhất
Việc triển khai đã nhấn mạnh một số bài học kinh nghiệm chính. Đầu tiên và quan trọng nhất, việc bắt đầu với các mục tiêu kinh doanh rõ ràng đã chứng tỏ là điều thiết yếu. Sự tập trung cụ thể của Empower vào việc tự động hóa các đánh giá GEDAC đã cung cấp các tiêu chí thành công có thể đo lường được, vốn đã định hướng cho mọi quyết định kỹ thuật. Chất lượng của đầu ra AI tương quan trực tiếp với chất lượng câu lệnh, khiến cho phương pháp tiếp cận lặp của Accenture đối với kỹ thuật tạo câu lệnh trở nên quan trọng để đạt được độ chính xác cần thiết cho việc triển khai sản xuất.

Lên kế hoạch cho quy mô ngay từ ngày đầu tiên là một yếu tố thành công quan trọng khác. Empower có thể xử lý khối lượng ngày càng tăng mà không cần thay đổi kiến trúc bằng cách triển khai sớm các khả năng suy luận liên khu vực và xử lý hàng loạt của Amazon Bedrock. Đội ngũ cũng nhận ra rằng AI tăng cường chứ không thay thế sự phán đoán của con người. Các chuyên viên phân tích chất lượng của Empower tiếp tục đóng một vai trò quan trọng trong việc xác thực các đánh giá của AI và xử lý các trườngChúng tôi nhận ra rằng để thực sự nâng cao trải nghiệm khách hàng trên quy mô lớn, chúng tôi cần phải tái định hình cơ bản phương pháp tiếp cận của mình đối với việc đảm bảo chất lượng,” Joe Mieras, Phó Giám đốc (VP) Dịch vụ Thành viên tại Empower, cho biết. “Quy trình thủ công đơn giản là không thể theo kịp tốc độ tăng trưởng và cam kết của chúng tôi về dịch vụ xuất sắc.”
trường hợp ngoại lệ, đảm bảo hệ thống duy trì các tiêu chuẩn chất lượng cao.

Cuối cùng, việc triển khai đã củng cố tầm quan trọng của việc cải tiến liên tục. Việc xem xét thường xuyên các đánh giá của AI giúp tinh chỉnh các câu lệnh và cải thiện độ chính xác theo thời gian, tạo ra một vòng lặp phản hồi đảm bảo hệ thống phát triển song song với nhu cầu kinh doanh. Phương pháp tiếp cận lặp này, kết hợp với quan hệ đối tác mạnh mẽ và các mục tiêu rõ ràng, đã tạo ra một kế hoạch chi tiết cho việc triển khai AI doanh nghiệp thành công, vượt ra ngoài một trường hợp sử dụng đơn lẻ này.

### Mở rộng tầm nhìn: Điều gì tiếp theo trên hành trình AI tạo sinh của Empower
Điều này chỉ là sự khởi đầu cho quá trình chuyển đổi AI tạo sinh của Empower. Dựa trên thành công này, Empower đang phát triển một nền tảng AI tạo sinh tập trung để dân chủ hóa các khả năng AI cho hơn 1.500 nhà phát triển của họ. Nền tảng này sẽ cung cấp quản trị tập trung để đảm bảo các biện pháp kiểm soát bảo mật, tuân thủ và AI có trách nhiệm nhất quán. Nó sẽ cung cấp một lớp trừu tượng để truy cập đơn giản hóa vào các mô hình và khả năng AI khác nhau, giám sát sử dụng toàn diện để theo dõi và quản lý chi phí, và chia sẻ các thông lệ tốt nhất với các thành phần có thể tái sử dụng nhằm tăng tốc độ phát triển giữa các nhóm.

<p align="center">
  <img src="https://d2908q01vomqb2.cloudfront.net/af3e133428b9e25c55bc59fe534248e6a0c0f17b/2025/07/30/connect-blog-image-4.png" 
       alt="Lớp Trừu tượng AI Tạo sinh của Empower" 
       width="50%">
  <br>
  <em>Lớp Trừu tượng AI Tạo sinh của Empower</em>
</p>


Empower đã xác định một số trường hợp sử dụng bổ sung để triển khai, trải rộng trên nhiều chức năng kinh doanh. Chúng bao gồm việc mở rộng đánh giá tự động sang các cuộc gọi tư vấn đầu tư và quản lý tài sản, đào tạo nhân viên bằng các trình mô phỏng hội thoại do AI cung cấp, tự động hóa việc phân tích tài liệu kế hoạch hưu trí, và nâng cao năng suất của nhà phát triển bằng các công cụ viết mã do AI cung cấp. Mỗi trường hợp sử dụng đều được xây dựng dựa trên nền tảng đã được thiết lập bởi giải pháp QA nhân viên, tận dụng các mẫu hình và quyết định kiến trúc đã được chứng minh.

Bằng cách kết hợp các dịch vụ AI mạnh mẽ có sẵn trong AWS, chuyên môn triển khai của Accenture và kiến thức chuyên môn về lĩnh vực của Empower, sự hợp tác này đã mang lại một giải pháp không chỉ đáp ứng các nhu cầu kinh doanh trước mắt mà còn đặt nền móng cho việc áp dụng AI rộng rãi hơn. Khi AI tạo sinh tiếp tục phát triển, những bài học này từ hành trình của Empower sẽ giúp các tổ chức khác định hướng cho quá trình chuyển đổi của riêng họ bằng cách bắt đầu với các mục tiêu rõ ràng, lựa chọn các đối tác phù hợp và duy trì sự tập trung không ngừng vào việc mang lại giá trị kinh doanh.