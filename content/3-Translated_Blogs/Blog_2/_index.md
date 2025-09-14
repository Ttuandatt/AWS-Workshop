+++
title = "Blog 2"
weight = 2
chapter = false
pre = " <b> 3.2. </b>"
+++

# Định tuyến động sử dụng Máy chủ định tuyến AWS VPC
[Amazon VPC Route Server](https://docs.aws.amazon.com/vpc/latest/userguide/dynamic-routing-route-server.html) cho phép triển khai cơ chế định tuyến động trong [Amazon VPC](https://aws.amazon.com/vpc/) bằng cách sử dụng giao thức BGP (Border Gateway Protocol). Với dịch vụ này, bạn có thể kiểm soát lưu lượng giữa các ứng dụng trên cloud và hệ thống on-premises một cách thông minh và hiệu quả hơn. BGP giúp bạn quản lý đường đi của dữ liệu linh hoạt, đặc biệt trong các tình huống xảy ra sự cố, đồng thời giảm thiểu khối lượng công việc thủ công và hạn chế rủi ro do con người.

Trong bài viết này, chúng ta sẽ xem xét nhiều tình huống ứng dụng trong đó việc định tuyến động ở mức ứng dụng có ảnh hưởng trực tiếp đến cách dữ liệu được truyền đến các instance, cũng như cách hệ thống xử lý khi có sự cố để giảm gián đoạn xuống mức thấp nhất.           


### Yêu cầu kiến thức nền tảng
Người đọc được giả định là đã nắm các khái niệm mạng cơ bản trong AWS liên quan đến tính sẵn sàng cao và cơ chế failover, chẳng hạn như [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/), [Elastic Network Interfaces (ENI)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html), Amazon VPC, bảng định tuyến VPC, và [Availability Zone (AZ)](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/). Ngoài ra, bạn cũng nên hiểu về các nguyên lý mạng phổ biến như địa chỉ IP, CIDR, định tuyến mạng, BGP và Bidirectional Forwarding Detection (BFD). Bài viết này sẽ không đi sâu vào việc giải thích các khái niệm nền tảng đó, mà tập trung mô tả cách sử dụng chúng để triển khai giải pháp floating IP cho việc chuyển đổi dự phòng ứng dụng. Nếu bạn cần tìm hiểu thêm về nền tảng mạng AWS, hãy tham khảo tài liệu chính thức của AWS về VPC networking và các bài viết trong mục AWS Networking and Content Delivery [Ở đây](https://docs.aws.amazon.com/vpc/latest/userguide/dynamic-routing-route-server.html).

### Các tình huống kết nối ứng dụng
Trong VPC, route table là thành phần quyết định luồng dữ liệu đi như thế nào. Chúng được gán cho subnet, Internet Gateway (IGW), hoặc Virtual Private Gateway để định nghĩa đường đi của lưu lượng trước khi đến đích. Ví dụ, bạn có thể thêm route để tất cả traffic đi qua một firewall trước khi đến ứng dụng, hoặc buộc subnet phải đi qua NAT Gateway, IGW, Peering Connection, hoặc VPN Gateway tùy nhu cầu.

Một số ứng dụng đặc biệt (như thiết bị an ninh mạng hoặc xử lý traffic) yêu cầu khả năng kiểm soát chi tiết đường đi của dữ liệu để đảm bảo tất cả lưu lượng phải được gửi đến ứng dụng trung gian này trước khi đến điểm đích cuối cùng. Một ví dụ quen thuộc là việc điều hướng traffic đến thiết bị bảo mật để kiểm tra trước khi gửi sang dịch vụ chính.

Nếu chỉ dùng static route thì bạn có thể làm được điều đó, nhưng nó tồn tại nhiều hạn chế:
- Không tự động thích ứng khi có sự cố.
- Cần thao tác thủ công để thay đổi.
- Quản lý trở nên phức tạp khi hệ thống mở rộng.
  Hệ quả là tăng nguy cơ lỗi con người và kéo dài thời gian phục hồi. Trong khi đó, dynamic routing có thể giải quyết: tự động cập nhật route table, đảm bảo khả năng mở rộng, và hỗ trợ failover mà không cần thao tác thủ công.

Lưu ý: AWS khuyến nghị sử dụng Gateway Load Balancer (GWLB) cho nhu cầu HA và redundancy. Chỉ khi nào ứng dụng không hỗ trợ GWLB (hoặc môi trường không có GWLB, ví dụ AWS Local Zone), bạn mới nên cân nhắc giải pháp với Route Server.

### Tính năng của VPC Route Server
VPC Route Server cung cấp định tuyến động bên trong VPC thông qua BGP. Các ứng dụng mạng có thể dùng BGP để cập nhật bảng định tuyến của VPC, từ đó kiểm soát lưu lượng chi tiết và hỗ trợ failover tự động giữa các instance trong cùng hoặc khác AZ. Route Server cũng có thể cập nhật bảng định tuyến của VPC và IGW với các đường đi IPv4/IPv6 ưu tiên, giúp workload có khả năng chịu lỗi cao. Khi một AZ gặp sự cố, Route Server sẽ tự động cập nhật route table để chuyển hướng traffic sang AZ khác, cải thiện khả năng quản lý và tính tương thích với ứng dụng của bên thứ ba.
Chúng ta sẽ xem xét các tình huống dưới đây để thấy được các khả năng định tuyến của VPC Route Server.

**Tình huống #1: Dùng Floating IP cho application failover (chuyển đổi dự phòng ứng dụng)**
Giả sử bạn có ứng dụng quan trọng chạy trên một EC2 ở AZ1. Để đảm bảo tính sẵn sàng, bạn triển khai thêm một instance standby ở AZ2. Ứng dụng này không tích hợp với GWLB hoặc môi trường không hỗ trợ GWLB. Mục tiêu của bạn là duy trì ứng dụng hoạt động liên tục khi instance chính hoặc cả AZ1 bị lỗi.

Trong kịch bản này, chúng ta minh họa cách sử dụng floating IP để thực hiện quá trình failover liền mạch giữa hai EC2 instance được triển khai ở hai AZ khác nhau trong một kiến trúc high availability. Bạn có một ứng dụng quan trọng chạy trên EC2 ở AZ1, đồng thời một EC2 khác được triển khai ở AZ2 để làm máy dự phòng. Ứng dụng này không được tích hợp với Gateway Load Balancer (GWLB), hoặc môi trường AWS của bạn (ví dụ Local Zone) không hỗ trợ GWLB. Mục tiêu đặt ra là đảm bảo cơ chế dự phòng cho ứng dụng trong trường hợp instance chính hoặc cả AZ1 gặp sự cố.

Bạn có thể dùng AWS CloudFormation từ repo aws-samples để triển khai kịch bản #1 vào tài khoản AWS của mình. Template CloudFormation sẽ tự động tạo ra môi trường sau:

- Một VPC với 3 subnet nằm trên 2 AZ.
- Bảng định tuyến (route table) cho cả 3 subnet.
- Tạo và gắn Internet Gateway (IGW) vào VPC, đồng thời thêm route mặc định ra IGW trong route table.
- Tạo và gắn Route Server cho VPC (Route Server sử dụng ASN 65000).
- Tạo hai Route Server Endpoint (RSE) trong mỗi subnet để đảm bảo tính sẵn sàng cao.
- Thiết lập các route server peer.
- Tạo hai EC2 instance để mô phỏng ứng dụng HA cần kiểm thử, sử dụng phần mềm Gobgp.
- Mỗi instance sẽ chạy BGP với ASN 65001 và kết nối BGP tới RSE trong subnet tương ứng.
- File cấu hình Gobgp (gobgpd.conf) được nạp sẵn thông qua user-data khi tạo EC2, và được lưu tại thư mục /home/ec2-user.
- Tạo thêm một instance test để ping tới loopback IP của ứng dụng HA.
- Có thể sử dụng AWS Systems Manager để truy cập và quản lý các instance vừa tạo.

**Tổng quan giải pháp**
Chúng ta sử dụng một địa chỉ floating IP được cấp từ dải CIDR nằm ngoài VPC để ứng dụng sử dụng. Client sẽ kết nối đến ứng dụng thông qua địa chỉ IP này. Trong trường hợp instance chính gặp sự cố, toàn bộ lưu lượng gửi đến floating IP sẽ được tự động chuyển hướng sang ENI của instance dự phòng ở AZ thứ hai. Cách làm này giúp giảm thiểu gián đoạn dịch vụ, nhờ áp dụng cơ chế floating IP kết hợp với định tuyến động trong VPC, mà không cần cập nhật cấu hình phía client hay thực hiện thao tác thủ công.

{{< figurecaption src="/images/Img1-Blog2.png" caption="Figure 1. Instance#1 is active" >}} <!-- cái này được config trong layouts/shortcodes/figurecaption.html để cho các dòng caption chú thích ảnh nằm ở giữa ảnh. Ta tạo thủ công layouts/shortcodes/figurecaption.html để config -->


Như minh họa ở Hình 1, ứng dụng hoạt động theo mô hình active/standby giữa hai AZ. Cả hai EC2 instance cùng quảng bá một địa chỉ loopback (ví dụ 172.16.1.1/32) ra mạng thông qua phiên BGP với hai VPC RSE nằm trong cùng một subnet. Việc sử dụng hai RSE nhằm đảm bảo tính dự phòng và nâng cao khả năng sẵn sàng của dịch vụ định tuyến.