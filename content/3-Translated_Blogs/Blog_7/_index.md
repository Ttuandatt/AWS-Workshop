+++
title = "Blog 7"
weight =  7
chapter = false
pre = " <b> 3.7. </b>"
+++

# Những hiểu biết sâu sắc và bài học kinh nghiệm từ Amazon Q trong tích hợp trình thu thập thông tin web Connect

Tác giả: Vikas Prasad & Ayush Mehta
Ngày: 22/07/2025

## Giới thiệu

Các nhân viên tổng đài (Human agents) là yếu tố then chốt trong mọi trung tâm chăm sóc khách hàng, và việc cung cấp cho họ các công cụ phù hợp để thành công là điều thiết yếu đối với mọi tổ chức.
Khi được hỗ trợ tốt, các nhân viên không chỉ có trải nghiệm làm việc tốt hơn, mà còn giúp nâng cao trải nghiệm của khách hàng cuối cùng.

Để đáp ứng nhu cầu này, Amazon Connect giới thiệu [Amazon Q in Connect](https://aws.amazon.com/connect/q/) — một trợ lý AI sinh nội dung (Generative AI assistant), cung cấp các phản hồi và hành động gợi ý theo thời gian thực, được cá nhân hóa giúp nhân viên tổng đài thực hiện công việc hiệu quả và nhanh chóng hơn.

Amazon Q in Connect có thể tích hợp với nhiều cơ sở tri thức (knowledge base) khác nhau thông qua các phương thức tích hợp đa dạng.
Bài viết này tập trung vào phương pháp tích hợp bằng Web crawler, bao gồm những lưu ý quan trọng khi triển khai và các thực hành tốt nhất.
Chúng ta sẽ cùng tìm hiểu cách khách hàng có thể truy cập và sử dụng hiệu quả các URL trang web của mình làm nguồn dữ liệu cho Amazon Q in Connect.

# Vì sao cần tích hợp Web Crawler?

Nội dung thường bị phân tán trên nhiều website, buộc agent phải tra cứu nhiều nguồn khác nhau khi hỗ trợ khách hàng. Điều này làm chậm quá trình xử lý và ảnh hưởng đến chất lượng tương tác.

- Khi được cấu hình với Web Crawler, Amazon Q in Connect có thể:

* Tự động thu thập và lập chỉ mục nội dung từ nhiều website, trang hỗ trợ.

- Cung cấp cho agent thông tin sản phẩm nhanh chóng và chính xác.

- Cung cấp liên kết nguồn gốc để agent và khách có thể đối chiếu.

- Loại bỏ việc cập nhật thủ công khi nội dung website thay đổi.

Bài viết này sẽ trình bày các giai đoạn và cách tối ưu triển khai Web Crawler trong Amazon Q in Connect.
![alt text](/aws/hinhanh/hinh1.jpg)

**Biểu đồ 1: Các giai đoạn khác nhau để tối ưu hóa việc triển khai trình thu thập thông tin web**

# 1. Giai đoạn Planning (Lập kế hoạch)

Trước khi bắt đầu triển khai tích hợp Web crawler cho Amazon Q in Connect, bạn nên tham khảo tài liệu hướng dẫn chính thức của [Amazon Connect](https://docs.aws.amazon.com/connect/latest/adminguide/enable-q.html) để biết chi tiết về cách kích hoạt [Amazon Q in Connect](https://docs.aws.amazon.com/connect/latest/adminguide/enable-q.html) cũng như các yêu cầu tiên quyết cần đáp ứng. Việc hiểu rõ nền tảng kỹ thuật và điều kiện cần thiết sẽ giúp quá trình triển khai diễn ra thuận lợi và tránh phát sinh lỗi khi cấu hình hoặc thu thập dữ liệu.

Trước khi triển khai tích hợp Web crawler, hãy xem xét kỹ ba nhóm câu hỏi nền tảng sau:

### Chiến lược quản lý tri thức

Trước hết, tổ chức cần xác định rõ bộ phận chịu trách nhiệm chính trong việc quản lý và duy trì nội dung — có thể là phòng Marketing, bộ phận IT, hoặc đội ngũ Trải nghiệm khách hàng (Customer Experience Team). Việc chỉ định rõ trá ch nhiệm giúp đảm bảo tính nhất quán trong quản trị dữ liệu. Đồng thời, cần thiết lập một quy trình cập nhật nội dung minh bạch, trong đó các thay đổi phải được đồng bộ hóa với hệ thống tổng đài Amazon Connect để các nhân viên luôn truy cập được phiên bản mới nhất của tri thức nội bộ.

### Phân tích người dùng cuối

Bước tiếp theo là xác định đối tượng sẽ trực tiếp sử dụng cơ sở tri thức. Họ có thể là các agent tuyến 1 — những người chủ yếu xử lý các câu hỏi tổng quát, hoặc các nhân viên hỗ trợ kỹ thuật chuyên sâu phụ trách những vấn đề phức tạp hơn. Trong nhiều trường hợp, hệ thống có thể được sử dụng bởi cả hai nhóm người dùng. Việc phân tích chính xác đặc điểm của người dùng cuối giúp doanh nghiệp xác định phạm vi nội dung cần crawl, cũng như độ sâu chi tiết của thông tin cần được lập chỉ mục để phù hợp với nhu cầu thực tế của từng nhóm.

### Khả năng truy cập nội dung

Cuối cùng, tổ chức cần kiểm tra khả năng truy cập của các nguồn dữ liệu được chọn để tích hợp vào Amazon Q in Connect. Một số nguồn thông tin có thể công khai, trong khi những nguồn khác yêu cầu xác thực hoặc quyền truy cập đặc biệt. Do đó, việc đảm bảo có sự cho phép hợp pháp trước khi tiến hành crawl là cực kỳ quan trọng. Điều này không chỉ tuân thủ các chính sách bảo mật và quyền riêng tư, mà còn giúp quá trình tích hợp diễn ra ổn định, bền vững và không bị gián đoạn bởi các vấn đề liên quan đến quyền truy cập.

# 2. Giai đoạn Staged Implementation (Triển khai theo giai đoạn)

Trong quá trình triển khai tích hợp Web crawler cho Amazon Q in Connect, doanh nghiệp nên bắt đầu với cách tiếp cận có trọng tâm và từng bước thay vì thực hiện đồng loạt trên toàn bộ hệ thống. Cụ thể, hãy tạo các cơ sở tri thức (knowledge base) riêng biệt cho từng nhóm nội dung quan trọng hoặc lĩnh vực trọng điểm. Việc này giúp dễ dàng quản lý, thử nghiệm và kiểm soát chất lượng dữ liệu của từng phần trước khi hợp nhất.

Tiếp theo, cần kiểm thử độc lập nhiều cấu hình URL khác nhau nhằm xác định đâu là phương án crawl hiệu quả nhất cho từng loại nội dung. Chỉ sau khi đã tối ưu từng phần, bạn mới nên kết hợp các cấu hình này thành một giải pháp tổng thể. Phương pháp này giúp bạn phân tích và xử lý lỗi (troubleshoot) đối với các nguồn nội dung cụ thể mà không ảnh hưởng đến toàn bộ hệ thống. Đồng thời, nó cho phép bạn tinh chỉnh mô hình quét (crawl pattern) để đạt được hiệu suất thu thập dữ liệu tối ưu.

Khi hệ thống bắt đầu hoạt động, các nhân viên tổng đài (agent) sẽ cung cấp phản hồi thực tế về chất lượng và mức độ phù hợp của các phản hồi do AI đề xuất. Dựa trên phản hồi này, bạn có thể mở rộng dần phạm vi crawl, bổ sung thêm các nhóm nội dung mới để hoàn thiện cơ sở tri thức.

Phương pháp triển khai theo giai đoạn không chỉ giúp rút ngắn thời gian triển khai ban đầu, mà còn đảm bảo mỗi phần nội dung được tối ưu kỹ lưỡng trước khi tích hợp vào cơ sở tri thức chính (primary knowledge base). Nhờ đó, hệ thống Amazon Q in Connect có thể đạt hiệu suất cao, dữ liệu nhất quán và độ tin cậy tối đa trong quá trình hỗ trợ nhân viên và khách hàng.

# 3. Giai đoạn Optimization (Tối ưu hóa)

Trong giai đoạn này, mục tiêu chính là tinh chỉnh cấu hình và chiến lược tích hợp web crawler để đạt hiệu suất cao nhất, đồng thời đảm bảo tính ổn định của hệ thống và chất lượng dữ liệu được thu thập. Dưới đây là các yếu tố thực tiễn quan trọng cần xem xét khi tối ưu hóa chiến lược tích hợp web crawler cho Amazon Q in Connect.

### Giới hạn dịch vụ (Service Limit)

Khi thiết kế hệ thống tích hợp web crawler cho Amazon Q, điều quan trọng là phải nhận thức rõ rằng mỗi crawler chỉ có thể xử lý tối đa [25.000 tệp (files)](https://docs.aws.amazon.com/bedrock/latest/userguide/webcrawl-data-source-connector.html#:~:text=Enter%20Maximum%20pages%20for%20data%20source%20sync%20between%201%20and%2025000.%20Limit) trong một phiên thu thập dữ liệu (ingestion).
Giới hạn này đòi hỏi doanh nghiệp phải có chiến lược chọn lọc URL và ưu tiên nội dung thật hiệu quả.

Cụ thể, nên xây dựng cơ chế lọc nội dung (content filtering mechanism) với thứ tự ưu tiên rõ ràng — tập trung vào:

- Các tài liệu quan trọng đối với hoạt động kinh doanh,

- Các bài viết thường xuyên được truy cập,

- Và tài nguyên hỗ trợ khách hàng có giá trị cao.

Bên cạnh đó, có thể áp dụng các kỹ thuật như nhận dạng mẫu URL (URL pattern matching), chấm điểm mức độ liên quan nội dung (content relevancy scoring) và lọc theo siêu dữ liệu (metadata filtering) để tận dụng tối đa giới hạn tệp cho phép.
Do hạn chế này, việc thiết kế kiến trúc nội dung hợp lý và phân tách cơ sở tri thức thành nhiều cấu hình crawler riêng biệt (multiple crawler configurations) là điều cần thiết trong một số trường hợp phức tạp.

### Tốc độ crawl (Crawl Rate)

Khi triển khai web crawler cho Amazon Q, doanh nghiệp cần phối hợp chặt chẽ với đội hạ tầng hoặc IT — những người trực tiếp quản lý website — để xác định ngưỡng tốc độ crawl (crawl rate threshold) phù hợp với khả năng tải và cấu hình cân bằng tải (load balancing) của hệ thống web.
Các yếu tố như số lượng yêu cầu đồng thời (concurrent requests) và tần suất crawl cần được hiệu chỉnh cẩn thận, vì việc quét quá nhanh có thể gây ra lỗi 504 Gateway Timeout và dẫn đến dữ liệu thu thập không đầy đủ.

Do đó, nên bắt đầu với tốc độ crawl thấp, sau đó tăng dần (progressive rate limit) trong khi theo dõi thời gian phản hồi của máy chủ web thông qua các công cụ giám sát lưu lượng truy cập (web traffic monitoring tools).
Bằng cách này, bạn có thể tinh chỉnh tham số crawl để vừa đảm bảo chỉ mục hóa dữ liệu chính xác và ổn định, vừa duy trì độ ổn định cho website.

### Định nghĩa phạm vi crawl (Crawl Scope)

Thay vì cố gắng crawl toàn bộ trang web, bạn nên xác định chính xác các điểm bắt đầu crawl (seed URLs) cho từng danh mục nội dung quan trọng.
Cách làm này giúp hệ thống khám phá nội dung liên quan một cách hiệu quả trong khi giảm số lượng seed URL cần thiết.
Ví dụ:

```bash
{ "url": "https://example.com/products/" },
{ "url": "https://example.com/documentation/" }

```

Trong ví dụ trên, mỗi seed URL trỏ đến trang danh mục chính (category index page) thay vì từng trang sản phẩm riêng lẻ, cho phép web crawler tự động tìm thấy toàn bộ nội dung liên quan mà không cần khai báo thủ công hàng trăm URL.

### Đánh giá chiến lược cơ sở tri thức – đơn hay đa

Tùy thuộc vào mô hình kinh doanh và đặc thù hỗ trợ khách hàng, bạn có thể chọn giữa:

- Một cơ sở tri thức thống nhất (unified knowledge base) — phù hợp khi khách hàng thường xuyên đặt câu hỏi liên quan đến nhiều sản phẩm khác nhau.

- Nhiều cơ sở tri thức tách biệt (separate knowledge bases) — phù hợp khi mỗi nhóm khách hàng chỉ quan tâm đến một dòng sản phẩm cụ thể.

Tuy nhiên, cần lưu ý đến giới hạn số lượng knowledge base mà dịch vụ Amazon Q cho phép.
Trong một số trường hợp nâng cao, bạn có thể thay đổi động (dynamic switching) giữa các knowledge base ngay trong quá trình chạy (runtime) bằng cách:

1. [Ghi đè cấu hình cơ sở tri](https://docs.aws.amazon.com/connect/latest/adminguide/create-ai-agents.html#cli-ai-agents-sample4) thức bằng AI Agent phù hợp.

2. Sau đó sử dụng API [Update session](https://docs.aws.amazon.com/connect/latest/APIReference/API_amazon-q-connect_UpdateSession.html) để chuyển đổi giữa các AI Agent tương ứng.

Cách tiếp cận linh hoạt này giúp Amazon Q in Connect có thể phục vụ nhiều loại khách hàng và tình huống hỗ trợ khác nhau mà không cần tái cấu hình toàn bộ hệ thống.
![alt text](/aws/hinhanh/hinh2.jpg)

**Sơ đồ 2: Các thành phần của Q trong Connect Assistant**

# 4. Giai đoạn Implementation (Triển khai thực tế)

### Giới hạn thời gian (Execution Timeout)

Khi triển khai web crawler của Amazon Q, các kiến trúc sư hệ thống cần tính đến giới hạn thời gian thực thi (timeout) cho mỗi phiên crawl.
Giới hạn này yêu cầu phải áp dụng chiến lược crawl hiệu quả, bao gồm:

- Ưu tiên URL (URL prioritization) để tập trung vào các trang quan trọng nhất.

- Tối ưu hóa mô hình crawl (optimized crawl patterns) nhằm giảm tải và tăng tốc độ thu thập dữ liệu.

- Và phân tách nội dung thành nhiều phiên crawl nhỏ (segmenting content into multiple crawl jobs) để đảm bảo hoàn thành quá trình thu thập dữ liệu trong khoảng thời gian cho phép.

### Giới hạn URL và cấu hình (URL Configuration)

Giao diện người dùng (UI) của Amazon Connect cho phép thêm tối đa 10 URL, trong khi API [UrlConfiguration](https://docs.aws.amazon.com/connect/latest/APIReference/API_amazon-q-connect_UrlConfiguration.html) hỗ trợ lên tới 100 URL khởi tạo (seed URLs).
Đối với các tổ chức có nhiều website hoặc lượng nội dung lớn, việc lập kế hoạch thông minh là rất cần thiết.
Thay vì liệt kê từng trang một, hãy xây dựng sơ đồ cấu trúc nội dung (content structure mapping) để xác định các danh mục chính (key categories).
Sau đó, tạo các biểu thức chính quy (regex patterns) để xác định:

- Nội dung cần thu thập (inclusion patterns),

- Và nội dung cần loại trừ (exclusion patterns).

Ví dụ:

```bash
._domain\.com/support/._\.pdf
```

# Kết luận

Tóm lại, thực tiễn tốt nhất (best practice) cho việc tích hợp trình thu thập dữ liệu web (web crawler integration) trong Amazon Q in Connect tuân theo một quy trình hệ thống gồm bốn giai đoạn chính:
Planning (Lập kế hoạch), Staged Implementation (Triển khai theo giai đoạn), Optimization (Tối ưu hóa) và Implementation (Triển khai thực tế).

**Giai đoạn Lập kế hoạch (Planning phase)**

Tập trung vào việc xây dựng chiến lược cơ sở tri thức (knowledge base strategy), phân tích nhu cầu người dùng cuối, và đảm bảo khả năng truy cập nội dung.
Việc chuẩn bị kỹ lưỡng ở giai đoạn này giúp định hướng chính xác cho các bước triển khai sau.

**Giai đoạn Triển khai theo giai đoạn (Staged Implementation phase)**
Ở bước này, tổ chức sẽ tạo và kiểm thử các cơ sở tri thức riêng biệt cho từng nhóm nội dung quan trọng, trước khi tích hợp chúng lại thành một hệ thống hoàn chỉnh.
Cách tiếp cận theo từng phần giúp kiểm soát chất lượng dữ liệu, xử lý lỗi hiệu quả, và đảm bảo độ chính xác của thông tin mà các agent sử dụng.

**Giai đoạn Tối ưu hóa (Optimization phase)**
Giai đoạn này tập trung giải quyết các yếu tố kỹ thuật then chốt như:
Giới hạn dịch vụ (service limits) — chẳng hạn giới hạn 25.000 tệp cho mỗi lần crawl.

Tốc độ thu thập dữ liệu (crawl rate) — để tránh gây tải cho máy chủ.

Và chiến lược cơ sở tri thức (knowledge base strategy) — xác định xem nên hợp nhất hay tách riêng cơ sở tri thức theo từng dòng sản phẩm.

**Giai đoạn Triển khai thực tế (Implementation phase)**
Xử lý các vấn đề thực thi như:
Giới hạn thời gian thu thập (execution timeout),

Và giới hạn cấu hình URL (URL configuration limits).
Tại đây, việc giám sát, tinh chỉnh và xác minh quá trình crawl là rất quan trọng để đảm bảo toàn bộ nội dung được thu thập chính xác và đầy đủ.

**Quy trình lặp lại (Iterative Process)**
Toàn bộ quy trình được thiết kế theo mô hình lặp (iterative), trong đó phản hồi giữa các giai đoạn được sử dụng để liên tục cải thiện và tối ưu hiệu suất tích hợp.
Cách tiếp cận có hệ thống này giúp các tổ chức triển khai Amazon Q’s web crawler một cách hiệu quả, đồng thời duy trì sự ổn định của hệ thống và đảm bảo bao quát toàn diện nguồn tri thức cho hoạt động của trung tâm liên hệ (contact center).

**Để đạt hiệu quả cao nhất khi sử dụng web crawler integration, cần lưu ý:**

1.Xác định nội dung ưu tiên cao (Identify high-priority content) và phân tích kỹ các URL.

2.Đặt các điểm bắt đầu crawl cụ thể (Set specific starting points for crawls) để giới hạn phạm vi hợp lý.

3.Sử dụng bộ lọc regex (Use regex filters) nhằm nhắm chính xác đến các URL có liên quan.

Theo dõi hiệu suất của Amazon Q in Connect qua Amazon [CloudWatch Logs](https://docs.aws.amazon.com/connect/latest/adminguide/monitor-q-assistants-cloudwatch.html), đảm bảo hệ thống hoạt động ổn định và hiệu quả.
Bằng cách áp dụng các chiến lược này, các tổ chức có thể tích hợp thành công dữ liệu tri thức từ nhiều website vào Amazon Q in Connect, giúp nhân viên hỗ trợ (agents) có quyền truy cập toàn diện vào thông tin sản phẩm và tài liệu hướng dẫn, đồng thời duy trì hoạt động thu thập dữ liệu hiệu quả trong giới hạn dịch vụ.
