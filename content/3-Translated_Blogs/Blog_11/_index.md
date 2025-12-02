+++
title = "Blog 11"
weight =  11
chapter = false
pre = " <b> 3.11. </b>"
+++

# Làm thế nào để quản lý Bot AI bằng AWS WAF và tăng cường bảo mật

**Tác giả:** [Kartik Bheemisetty](https://aws.amazon.com/), và [David MacDonald](https://aws.amazon.com/)  

**Ngày:** 01 tháng 8 năm 2025

**Thể loại:** AWS WAF, Thông lệ tốt nhất, Bảo mật, Danh tính, & Tuân thủ, Hướng dẫn kỹ thuật, Lãnh đạo tư tưởng

---


### Giới thiệu

Trình thu thập dữ liệu web (web crawler) đầu tiên được tạo ra vào năm 1993 để đo lường kích thước của web, và giờ đây chúng đã phát triển thành các bot hiện đại được hỗ trợ bởi AI tự hành. Internet ngày nay ngày càng có nhiều và bị thống trị bởi các bot AI tự động tương tác với các ứng dụng để hỗ trợ các tác vụ liên quan đến AI.

Chúng tôi phân loại bot AI thành ba loại:

- **Bot AI cào dữ liệu (AI scrapers)**, loại bot này thu thập dữ liệu một cách có hệ thống từ ứng dụng của bạn để huấn luyện các mô hình AI.  
- **Công cụ AI (AI tools)**, loại này hiển thị dữ liệu từ ứng dụng của bạn trong các ứng dụng AI bằng cách sử dụng [tính năng gọi hàm (Function calling)](https://docs.aws.amazon.com/bedrock/latest/userguide/tool-use.html).  
- **Tác tử AI (AI agents)**, loại này có thể tự chủ điều hướng và tương tác động với ứng dụng của bạn để thực hiện các tác vụ phức tạp.  

Mặc dù một số bot AI cung cấp các dịch vụ có giá trị như tự động hóa các tác vụ tẻ nhạt, nhưng một số bot độc hại có thể gây ra những thách thức đáng kể cho chủ sở hữu và người vận hành ứng dụng web. Các bot độc hại có thể làm quá tải máy chủ với lưu lượng truy cập quá mức, dẫn đến suy giảm hiệu suất hoặc thậm chí ngừng hoạt động. Nếu không được kiểm soát, các bot này không chỉ làm tổn hại đến bảo mật mà còn có thể làm xói mòn lòng tin của người dùng và làm tổn hại danh tiếng thương hiệu.

Trong bài viết này, chúng tôi khám phá các vấn đề khác nhau do bot AI gây ra và tìm hiểu các cơ chế khác nhau để phát hiện và quản lý bot AI bằng cách sử dụng [Amazon Web Services (AWS) WAF](https://aws.amazon.com/waf/).

---

### Điều kiện tiên quyết

Bài viết này tập trung vào [AWS WAF](https://aws.amazon.com/waf/) như là tuyến phòng thủ đầu tiên để quan sát và quản lý hoạt động của bot AI nhắm vào ứng dụng của bạn.  
Nếu bạn chưa kích hoạt bảo vệ bằng AWS WAF, thì bạn có thể bắt đầu bằng cách trực quan hóa bối cảnh mối đe dọa bằng [AWS Shield network security director](https://docs.aws.amazon.com/waf/latest/developerguide/nsd-chapter.html). Nó giúp bạn xác định các tài nguyên không được bảo vệ bằng AWS WAF.

Sau đó, bạn có thể bắt đầu bằng cách tạo một tư thế bảo mật ban đầu bằng cách sử dụng các tích hợp bảo mật chỉ bằng một cú nhấp chuột. Nó tự động tạo ra một [gói bảo vệ hoặc web ACL](https://docs.aws.amazon.com/waf/latest/developerguide/web-acl.html) với các quy tắc để bảo vệ ứng dụng của bạn khỏi các mối đe dọa phổ biến nhất. Xem các tài liệu tham khảo sau:

- Nếu bạn đang sử dụng [Amazon CloudFront](https://aws.amazon.com/cloudfront/) để lưu trữ ứng dụng của mình, thì hãy kích hoạt bảo vệ bằng [tích hợp AWS WAF một cú nhấp chuột của CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/WAF-one-click.html).  
- Nếu bạn đang sử dụng [Application Load Balancer (ALB)](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) để lưu trữ ứng dụng của mình, thì hãy kích hoạt bảo vệ bằng [tích hợp AWS WAF một cú nhấp chuột của ALB](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/application-load-balancers.html#load-balancer-waf).


### Các vấn đề do bot AI gây ra

Bot không phải là một mối đe dọa mới trên web. Tuy nhiên, các yêu cầu về dữ liệu của **Mô hình Ngôn ngữ Lớn (LLMs)** và các mẫu tương tác mới được kích hoạt bởi **tác tử AI** đã khiến hành vi của bot trở nên rắc rối hơn trên nhiều ứng dụng.  
Các ứng dụng web có thể phải đối mặt với các vấn đề sau đây do bot AI gây ra:

1. **Sử dụng dữ liệu độc quyền để huấn luyện mô hình:** Việc sử dụng trái phép dữ liệu của tổ chức bạn có thể tạo ra các lo ngại về sở hữu trí tuệ khi chúng được sử dụng để huấn luyện các mô hình AI.  
   Ví dụ, nội dung của bạn có thể được sử dụng để tạo ra các dịch vụ có khả năng cạnh tranh mà không có sự đền bù và làm loãng giá trị thị trường độc nhất của nội dung.

2. **Hiệu suất kém và chi phí cao:** Các bot AI đang thu thập nội dung ứng dụng của bạn một cách ráo riết có thể tạo ra lưu lượng truy cập quá tải, dẫn đến suy giảm hiệu suất đối với người dùng hợp lệ.  
   Điều này cũng có thể phát sinh chi phí **truyền dữ liệu ra ngoài (DTO)**, gây lãng phí tài nguyên tính toán và có khả năng gây ngừng dịch vụ trong các thời kỳ cào dữ liệu cao điểm.

3. **Hành vi tự động/tự hành không mong muốn:** Các bot AI có thể tự động tương tác với ứng dụng của bạn mà không cần sự tham gia của con người.  
   Điều này có thể lấy đi lưu lượng truy cập giá trị của con người khỏi ứng dụng của bạn vì AI có thể tóm tắt các phát hiện của nó.  
   Các bot AI cũng có thể cạnh tranh với người dùng hợp lệ để hoàn thành các **quy trình công việc có giá trị cao, nhạy cảm về thời gian** như mua hàng tồn kho có hạn.  

   Các bot này thường sử dụng các kỹ thuật sau để tương tác với ứng dụng của bạn:

   - **Gọi hàm (Function calling) và tìm kiếm AI:** Các ứng dụng AI sử dụng các công cụ để tìm kiếm và đưa ra các yêu cầu dữ liệu một lần trực tiếp từ ứng dụng của bạn.

   - **Tương tác qua Khuôn khổ Tự động hóa Trình duyệt:** Các tác tử AI như Amazon Nova Act sử dụng Playwright để điều khiển các trình duyệt thực.  
     Chúng có thể hoàn thành các tác vụ nhiều bước và tương tác với các ứng dụng theo cách giống như con người. Các tác tử này có thể thực thi JavaScript và xử lý các yếu tố UI phức tạp một cách hiệu quả.

   - **Tương tác dựa trên VM:** Các hệ thống như Computer Use của Anthropic hoạt động trong môi trường máy ảo (VM). Chúng tương tác với các ứng dụng theo cách giống con người hơn. Không giống như các trình duyệt tự động của Playwright, các hệ thống này sử dụng các cài đặt trình duyệt tiêu chuẩn, khiến cho hành vi của chúng gần như không thể phân biệt được với người dùng thực.

---

### Xác định quy mô của hoạt động bot AI

Đầu tiên, bạn cần hiểu bot AI ảnh hưởng đến ứng dụng của bạn như thế nào và ở quy mô nào.  
Bắt đầu bằng cách thêm nhóm quy tắc AWS WAF Bot Control vào gói bảo vệ tài nguyên của bạn với Cấp độ **Kiểm tra Chung (Common Inspection)**.  
Sử dụng **Chế độ Đếm (Count mode)** ban đầu để giám sát các mẫu lưu lượng truy cập. Phương pháp tiếp cận này cho phép bạn phân tích hoạt động của bot trước khi thực hiện các thay đổi có thể ảnh hưởng đến lưu lượng truy cập sản xuất.

Nhóm quy tắc chung của Bot Control xác minh các bot tự nhận dạng thông qua xác thực chữ ký.  
Nó bao gồm một quy tắc **CategoryAI** để phát hiện các bot AI đã được xác minh.  
Hãy chắc chắn rằng bạn cấu hình nhóm quy tắc với **phiên bản mới nhất**, như được hiển thị trong *Hình 1* sau đây.

<p align="center">
  <img src="https://d2908q01vomqb2.cloudfront.net/5b384ce32d8cdef02bc3a139d4cac0a22bb029e8/2025/08/01/NetCDNBlog-1534-1.jpeg" 
       alt="Hình 1: Nhóm quy tắc AWS WAF Bot Control với mức kiểm tra “Common” và phiên bản 3.2" 
       width="50%">
  <br>
  <em>Hình 1: Nhóm quy tắc AWS WAF Bot Control với mức kiểm tra “Common” và phiên bản 3.2</em>
</p>

Sau khi chạy nhóm quy tắc được quản lý trong vài ngày, bạn có thể phân tích dữ liệu đã thu thập. Để xem các thông tin chi tiết, hãy mở [bảng điều khiển AWS WAF và AWS Shield](https://console.aws.amazon.com/wafv2/home) và chọn Khu vực AWS của bạn. Chọn gói bảo vệ của bạn và chọn xem bảng điều khiển. Điều hướng đến phần Tổng quan và chọn tùy chọn Bots để xem hoạt động, phát hiện, danh mục và tín hiệu của bot. Bảng điều khiển này cung cấp thông tin chi tiết về hoạt động của bot trên ứng dụng của bạn.

Hình 2 sau đây cho thấy một ví dụ về phần Các danh mục Bot. Nó hiển thị một khối lượng lớn các yêu cầu được đánh dấu là **ai - AllowedRequests**. Đây là các bot AI được quy tắc **CategoryAI** xác định, nhưng không bị chặn. Bạn cũng có thể nhận thấy các bot khác đang gửi khối lượng lớn yêu cầu.

<p align="center">
  <img src="https://d2908q01vomqb2.cloudfront.net/5b384ce32d8cdef02bc3a139d4cac0a22bb029e8/2025/08/01/NetCDNBlog-1534-2.jpg" 
       alt="Hình 2: Tổng quan lưu lượng truy cập cho quy tắc CategoryAI được AWS WAF phát hiện" 
       width="50%">
  <br>
  <em>Hình 2: Tổng quan lưu lượng truy cập cho quy tắc CategoryAI được AWS WAF phát hiện</em>
</p>

### Quản lý các vấn đề do bot AI gây ra

Trong các phần sau, chúng ta sẽ xem xét các phương pháp khác nhau để quản lý các vấn đề do bot AI gây ra.

---

### Chặn bot AI sớm bằng robots.txt

#### Kịch bản 1: Chặn sớm các bot AI tuân thủ quy tắc
Một tệp `robots.txt` giúp kiểm soát quyền truy cập của bot vào ứng dụng của bạn. Tệp văn bản đơn giản này được đặt tại thư mục gốc của ứng dụng (`/robots.txt`). Nó sử dụng một định dạng tiêu chuẩn để hướng dẫn các bot tuân thủ về những phần nào của ứng dụng mà chúng có thể và không thể truy cập. Mặc dù không phải tất cả các bot đều tuân theo các quy tắc này, nhưng các nhà điều hành bot có uy tín đều tôn trọng các tệp `robots.txt` được cấu hình đúng cách.  

Các dự án nguồn mở như **[ai.robots.txt](https://github.com/ai-robots-txt/ai.robots.txt)** cung cấp một tệp `robots.txt` với các trình thu thập dữ liệu (crawlers) liên quan đến AI mới nhất mà bạn có thể sử dụng để chặn các bot này trước khi chúng bắt đầu thu thập dữ liệu ứng dụng của bạn.

Nếu **[AWS WAF](https://aws.amazon.com/waf/)** hiển thị khối lượng yêu cầu cao từ các bot cụ thể, thì bạn có thể sử dụng `robots.txt` để chặn các bot cào dữ liệu (scraping bots) quá mức nhưng tuân thủ quy tắc. Điều này giúp ngăn chúng ảnh hưởng đến DTO và hiệu suất ứng dụng của bạn.

Sau đây là một ví dụ cho phép **Amazonbot** thu thập dữ liệu các URL `/public` nhưng không thu thập dữ liệu các URL `/private`:

```bash
User-agent: Amazonbot 
Disallow: /private/ 
Allow: /public/
```

---

#### Kịch bản 2: Quản lý cách bot AI sử dụng dữ liệu của bạn
Các bot từ các công ty công nghệ lớn phục vụ mục đích kép: chúng cào dữ liệu ứng dụng của bạn một lần và sử dụng dữ liệu đó để lập chỉ mục tìm kiếm và huấn luyện các mô hình AI. Bạn có thể cho phép các bot này thu thập dữ liệu ứng dụng của bạn để lập chỉ mục tìm kiếm, trong khi vẫn yêu cầu chúng không sử dụng dữ liệu này để huấn luyện các LLM.  

Sau đây là ba ví dụ chỉ ra cách ngăn chặn các nhà điều hành bot lớn sử dụng dữ liệu của bạn để huấn luyện các LLM:

---

**1. Amazonbot:** Nó sử dụng tiêu đề phản hồi HTTP `X-Robots-Tag: noarchive` để báo hiệu rằng bạn không muốn phản hồi này được sử dụng để huấn luyện các LLM.  
Bạn có thể triển khai điều này bằng cách sử dụng **[chính sách tiêu đề phản hồi của CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/response-headers-policies.html)** để thêm tiêu đề này vào mọi phản hồi từ ứng dụng của bạn.

```bash
HTTP/1.1 200 OK 
Date: Tue, 15 Oct 2024 08:09:00 GMT 
X-Robots-Tag: noarchive
```

---

**2. Applebot:** Bạn có thể yêu cầu Apple không sử dụng dữ liệu từ ứng dụng của bạn để huấn luyện các mô hình học máy (ML) của họ bằng cách thêm một mục `User-agent Applebot-Extended` vào tệp `robots.txt` của bạn.  
Điều này vẫn cho phép Apple lập chỉ mục nội dung của bạn để tìm kiếm. Sau đây là một mục ví dụ để không cho phép Applebot-Extended (truy cập) trên toàn bộ ứng dụng của bạn:

```bash
User-agent: Applebot-Extended 
Disallow: /
```

Chỉ thị `User-agent` trong `robots.txt` phục vụ một mục đích cụ thể. Nó so khớp các mẫu với danh tính được khai báo của bot, vốn khác với tiêu đề `HTTP User-Agent`.

---

**3. Googlebot:** Tương tự, Google cho phép bạn không cho phép huấn luyện các mô hình ML của Google bằng cách thêm User Agent `Google-Extended` vào tệp `robots.txt` của bạn:

```bash
User-agent: Google-Extended 
Disallow: /
```

Một số nhà điều hành bot có thể không tôn trọng tệp `robots.txt`, vì vậy bạn phải cần quản lý chúng bằng **[AWS WAF](https://aws.amazon.com/waf/)**.


### Sử dụng AWS WAF

#### Kịch bản 3: Quản lý các bot AI đang gây ra hiệu suất kém và chi phí cao
Các bot AI cào dữ liệu ráo riết từ ứng dụng của bạn có thể làm suy giảm hiệu suất ứng dụng và phát sinh chi phí DTO cũng như tính toán cao. Bạn có thể sử dụng các kỹ thuật sau đây bằng **[AWS WAF](https://aws.amazon.com/waf/)** để bảo vệ ứng dụng của mình khỏi các bot không tôn trọng `robots.txt`:

---

1. **Quản lý các bot AI tự nhận dạng bằng nhóm quy tắc AWS WAF Bot Control với Cấp độ Kiểm tra Chung:** Bạn có thể chặn các yêu cầu bot AI có khối lượng lớn bằng cách xóa hành động **Ghi đè tất cả quy tắc** mà trước đó bạn đã đặt thành *Đếm* trong **nhóm quy tắc AWS WAF Bot Control** với *Cấp độ Kiểm tra Chung*.  
Quy tắc `CategoryAI` giờ đây sẽ chặn các yêu cầu bot AI này theo mặc định.

Ngoài các bot AI thuộc quy tắc `CategoryAI`, **AWS WAF** không chặn các bot phổ biến và có thể xác minh được.  
Nếu bạn xác định một bot đã được xác minh, hoặc một danh mục bot, vẫn đang tạo ra khối lượng truy cập cao, thì bạn phải thêm một quy tắc một cách rõ ràng **sau** **nhóm quy tắc AWS WAF Bot Control**.  

Quy tắc này sẽ chặn bot cụ thể (hoặc nhóm bot được đại diện bởi một không gian tên nhãn), như được hiển thị trong *hình 3* sau đây.


<p align="center">
  <img src="https://d2908q01vomqb2.cloudfront.net/5b384ce32d8cdef02bc3a139d4cac0a22bb029e8/2025/08/01/NetCDNBlog-1534-3.jpg" 
       alt="Hình 3: Quy tắc tùy chỉnh AWS WAF để chặn yandexbot bằng cách sử dụng nhãn (labels)" 
       width="50%">
  <br>
  <em>Hình 3: Quy tắc tùy chỉnh AWS WAF để chặn yandexbot bằng cách sử dụng nhãn (labels)</em>
</p>

2. **Làm chậm các bot cào dữ liệu lẩn tránh:** Các bot giả mạo tiêu đề HTTP user agent của chúng để giả vờ là các bot nổi tiếng, hoặc các máy khách hợp lệ của người dùng. Bạn có thể ngăn chặn các bot này làm quá tải ứng dụng của mình bằng cách sử dụng **tính năng bảo vệ DDoS lớp ứng dụng (L7) nâng cao của AWS WAF** và cả **quy tắc dựa trên tỷ lệ của AWS WAF**. Các quy tắc DDoS và quy tắc giới hạn tỷ lệ bảo vệ ứng dụng của bạn khỏi bất kỳ nguồn nào tạo ra khối lượng yêu cầu cao, bao gồm cả bot. Để tìm hiểu cách xác định ngưỡng quy tắc dựa trên tỷ lệ và các thông lệ tốt nhất về việc tạo quy tắc dựa trên tỷ lệ, hãy tham khảo bài viết **"[Ba quy tắc dựa trên tỷ lệ quan trọng nhất của AWS WAF](https://aws.amazon.com/blogs/security/three-most-important-aws-waf-rate-based-rules/)"**.

3. **Bắt các bot cào dữ liệu lẩn tránh phải làm việc:** Hành động **Thử thách (Challenge) của AWS WAF** chạy một thử thách ngầm trong môi trường máy khách mà không yêu cầu người dùng tương tác và được thiết kế để không có tác động rõ rệt đến trải nghiệm của người dùng. Thử thách yêu cầu máy khách hoàn thành một tác vụ tốn kém về mặt tính toán (bằng chứng công việc). Phương pháp tiếp cận này nhằm cung cấp cho người dùng hợp lệ một cơ chế liền mạch để xác thực môi trường của họ, đồng thời tăng chi phí cho các nhà điều hành bot đang cố gắng tương tác với ứng dụng của bạn.  

Hình 4 sau đây chỉ ra cách thêm một quy tắc tùy chỉnh sau **nhóm quy tắc AWS WAF Bot Control**. Quy tắc này yêu cầu người dùng phải hoàn thành một thử thách trước khi tiếp tục, trừ khi họ là các bot được phép/đã xác minh. Các bot đã xác minh được xác định bằng cách có một nhãn trong không gian tên `awswaf:managed:aws:bot-control:bot`.

<p align="center">
  <img src="https://d2908q01vomqb2.cloudfront.net/5b384ce32d8cdef02bc3a139d4cac0a22bb029e8/2025/08/01/NetCDNBlog-1534-4.jpg" 
       alt="Hình 4: Quy tắc AWS WAF buộc thực hiện thử thách đối với toàn bộ lưu lượng bot chưa được xác minh" 
       width="50%">
  <br>
  <em>Hình 4: Quy tắc AWS WAF buộc thực hiện thử thách đối với toàn bộ lưu lượng bot chưa được xác minh</em>
</p>


**4. Sử dụng honeypot để bẫy các bot lẩn tránh**

**[Giải pháp Tự động hóa Bảo mật cho AWS WAF](https://aws.amazon.com/solutions/implementations/aws-waf-security-automations/)** bao gồm một **điểm cuối honeypot** dụ các bot đang cào dữ liệu ứng dụng của bạn đến một điểm cuối mà không người dùng hợp lệ hoặc bot tuân thủ quy tắc nào truy cập.  
Điểm cuối này giúp **phát hiện và chặn** các IP độc hại, hạn chế hiệu quả tác động của các bot cào dữ liệu ứng dụng của bạn.

-> Xem video sau để hiểu rõ hơn
 <iframe width="560" height="315" src="https://www.youtube.com/embed/2mlQL2cQfDY?si=zI3RyAYKjpGVVeHK" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

**Kịch bản 4: Quản lý các bot AI tự hành/tự động không mong muốn**

Bạn có thể sử dụng các kỹ thuật sau để quản lý các bot AI tự hành:

**Nhóm quy tắc AWS WAF Bot Control với Cấp độ Kiểm tra Chung:** Quy tắc **CategoryAI** bao gồm các quy tắc cho các tác tử AI được xác định rõ ràng như **Amazon Nova Act**. Hơn nữa, các quy tắc **SignalNonBrowserUserAgent** và **SignalAutomatedBrowser** sẽ chặn **các tác tử tự động hóa trình duyệt kiểu Playwright**.

**Nhóm quy tắc AWS WAF Bot Control với Cấp độ Kiểm tra Mục tiêu:** Cấp độ kiểm tra này tạo ra một đường cơ sở thông minh về các mẫu lưu lượng truy cập. Cấp độ này sử dụng các kỹ thuật lấy dấu vân tay để giúp bảo vệ ứng dụng của bạn khỏi các bot tự hành bắt chước con người. Tham khảo bài viết [phát hiện và chặn lưu lượng truy cập bot nâng cao](https://aws.amazon.com/blogs/security/detect-and-block-advanced-bot-traffic-with-aws-waf-bot-control/) để được hướng dẫn thiết lập tính năng này.

**Hành động CAPTCHA của AWS WAF:** Các LLM từ các nhà cung cấp lớn được huấn luyện để không giải CAPTCHA. Điều này sẽ ngăn chặn nhiều tác tử hoàn thành tương tác được yêu cầu. Tương tự như kỹ thuật Thử thách trước đó, bạn có thể cấu hình một quy tắc với hành động CAPTCHA để yêu cầu (hoàn thành) đối với một số yêu cầu nhất định. Tham khảo bài viết [Sử dụng AWS WAF CAPTCHA để bảo vệ ứng dụng của bạn khỏi lưu lượng truy cập bot phổ biến](https://aws.amazon.com/blogs/security/use-aws-waf-captcha-to-protect-your-application-against-common-bot-traffic/) để được hướng dẫn thiết lập tính năng này.
  
**Xác thực (bao gồm cả sinh trắc học):** Cuối cùng, các bot sẽ tiếp tục cải thiện và lẩn tránh các biện pháp giảm thiểu. Nếu bạn có yêu cầu cao về tương tác của con người, thì hãy cân nhắc sửdụng xác thực, bao gồm cả sinh trắc học, trước khi tiếp tục tương tác. Tham khảo bài viết [Cách sử dụng AWS WAF Bot Control cho các tín hiệu bot được nhắm mục tiêu và giảm thiểu các bot lẩn tránh bằng trải nghiệm người dùng thích ứng](https://aws.amazon.com/blogs/security/how-to-use-aws-waf-bot-control-for-targeted-bots-signals-and-mitigate-evasive-bots-with-adaptive-user-experience/) để được hướng dẫn về cách thúc đẩy xác thực người dùng thích ứng khi các tương tác cho thấy có khả năng là lưu lượng truy cập bot.

### Kết luận:
Các bot AI tạo ra những thách thức đáng kể thông qua việc cào dữ liệu quá mức làm suy giảm hiệu suất và tăng chi phí, việc sử dụng nội dung trái phép để huấn luyện AI, và các tương tác tự động có thể từ gây phiền toái đến độc hại. Bằng cách triển khai các chiến lược đã được thảo luận trong bài viết này, bắt đầu từ các cấu hình **robots.txt** cơ bản đến các quy tắc AWS WAF Bot Control nâng cao, giới hạn tỷ lệ, và các thử thách CAPTCHA, bạn có thể bảo vệ khỏi việc cào dữ liệu trái phép, ngăn chặn suy giảm hiệu suất, và duy trì quyền kiểm soát đối với cách nội dung của bạn được các bot AI sử dụng.

Hơn nữa, để luôn được cập nhật về AWS WAF, hãy tham khảo [Blog Bảo mật AWS WAF](https://aws.amazon.com/blogs/security/tag/aws-waf/) và [Có gì mới với Bảo mật, Danh tính và Tuân thủ của AWS](https://aws.amazon.com/about-aws/).

Cảm ơn bạn đã đọc bài viết này. Nếu bạn có phản hồi về bài viết này, hãy gửi bình luận trong phần bình luận. Nếu bạn có câu hỏi về bài viết này, hãy bắt đầu một chủ đề mới trên [AWS WAF re:Post](https://repost.aws/tags/TAKdJ093DSSdGOQ1VVKX4EvQ/aws-waf) hoặc liên hệ [Hỗ trợ của AWS](https://aws.amazon.com/support/).