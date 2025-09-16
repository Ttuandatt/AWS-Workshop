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

{{< figurecaption src="/images/Img1-Blog2.png" caption="Hình 1. Instance#1 đang hoạt động" >}} <!-- cái này được config trong layouts/shortcodes/figurecaption.html để cho các dòng caption chú thích ảnh nằm ở giữa ảnh. Ta tạo thủ công layouts/shortcodes/figurecaption.html để config -->


Như minh họa ở Hình 1, ứng dụng hoạt động theo mô hình active/standby giữa hai AZ. Cả hai EC2 instance cùng quảng bá một địa chỉ loopback (ví dụ 172.16.1.1/32) ra mạng thông qua phiên BGP với hai VPC RSE nằm trong cùng một subnet. Việc sử dụng hai RSE nhằm đảm bảo tính dự phòng và nâng cao khả năng sẵn sàng của dịch vụ định tuyến.

{{< figurecaption src="/images/Img2-Blog2.png" caption="Hình 2. Các điểm cuối (endpoint) của máy chủ định tuyến trong VPC" >}}

{{< figurecaption src="/images/Img3-Blog2.png" caption="Hình 3. Các thiết bị ngang hàng của máy chủ định tuyến trong VPC" >}}

{{< figurecaption src="/images/Img4-Blog2.png" caption="Hình 4. Bảng RIB (Routing Information Base) của máy chủ định tuyến" >}}


Để đảm bảo lưu lượng mạng luôn được chuyển đến instance đang hoạt động, ứng dụng sử dụng thuộc tính BGP AS Path. Instance hoạt động sẽ quảng bá tuyến với AS Path ngắn hơn, trong khi instance dự phòng thêm nhiều số AS khác vào đường đi, khiến tuyến đó bị ưu tiên thấp hơn. Do BGP luôn chọn tuyến có AS Path ngắn nhất, nên tuyến từ instance hoạt động sẽ được chọn làm tuyến chính. Ngoài ra, các thuộc tính khác của BGP như Multi-Exit Discriminator (MED) cũng có thể được dùng để thiết lập ưu tiên định tuyến tương tự.
1. Cả instance hoạt động và dự phòng đều quảng bá địa chỉ 172.16.1.1/32 qua BGP đến cả hai Route Server Endpoint (RSE) trong subnet của chúng.

2. Máy chủ định tuyến VPC nhận được 4 quảng bá cho cùng một tiền tố. Điều này được minh họa ở Hình 4, nơi loopback 172.16.1.1/32 được gửi đến từ cả bốn RSE.

3. Dựa vào quy tắc chọn đường đi của BGP, máy chủ ưu tiên tuyến từ instance hoạt động vì có AS Path ngắn hơn. Trong Hình 4 thể hiện rõ rằng chỉ một trong bốn tuyến được chọn và cài đặt.

4. Máy chủ định tuyến VPC sau đó xác định ENI gắn với instance hoạt động và cập nhật bảng định tuyến của VPC để chuyển tiếp lưu lượng tới ENI-A cho địa chỉ 172.16.1.1/32.

{{< figurecaption src="/images/Img5-Blog2.png" caption="Hình 5. Bảng định tuyến VPC được cập nhật, trong đó tuyến 172.16.1.1/32 trỏ đến ENI của instance đang hoạt động." >}}

Bạn có thể kiểm tra cấu hình Gobgp bằng cách kết nối vào một trong các instance (instance-rs-az1 hoặc instance-rs-az2) thông qua [EC2 Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html).

Tệp cấu hình Gobgp nằm tại /home/ec2-user/gobgpd.conf.

```bash
sh-5.2$ sudo more /home/ec2-user/gobgpd.conf

[global.config]
as = 65001
router-id = "10.0.1.203"
[[neighbors]]
[neighbors.config]
neighbor-address = "10.0.1.230"
peer-as = 65000
[[neighbors.afi-safis]]
[neighbors.afi-safis.config]
afi-safi-name = "ipv4-unicast"
[[neighbors]]
[neighbors.config]
neighbor-address = "10.0.1.136"
peer-as = 65000
[[neighbors.afi-safis]]
[neighbors.afi-safis.config]
afi-safi-name = "ipv4-unicast"
```

Sử dụng lệnh sau để kiểm tra trạng thái BGP neighbor. Kết quả sẽ hiển thị hai neighbor, tương ứng với hai VPC RSE trong subnet của instance.

```bash
sh-5.2$ sudo /home/ec2-user/gobgp neighbor
Peer            AS     Up/Down   State   |#Received Accepted
10.0.1.136      65000  22:43:07  Establ  | 0        0
10.0.1.230      65000  22:43:08  Establ  | 0        0
```


Kiểm tra rằng tuyến loopback đã được quảng bá qua BGP:

```bash
sh-5.2$ sudo /home/ec2-user/gobgp global rib
   Network      Next Hop  AS_PATH  Age       Attrs
*> 172.16.1.1/32 0.0.0.0           22:42:21  [{Origin: ?}]
```

Để kiểm tra thiết lập định tuyến, bạn có thể truy cập vào instance kiểm thử (test-instance) bằng phương pháp Systems Manager. Sau khi đăng nhập thành công, hãy ping đến địa chỉ 172.16.1.1. Bạn sẽ nhận được phản hồi xuất phát từ instance đang hoạt động “instance-rs-az1”.

```bash
sh-5.2$ ping 172.16.1.1
PING 172.16.1.1 (172.16.1.1) 56(84) bytes of data.
64 bytes from 172.16.1.1: icmp_seq=1 ttl=127 time=0.712 ms
64 bytes from 172.16.1.1: icmp_seq=2 ttl=127 time=0.338 ms
64 bytes from 172.16.1.1: icmp_seq=3 ttl=127 time=0.378 ms
```

### Phát hiện và khôi phục khi xảy ra Failover

Để mô phỏng một tình huống failover, bạn có thể tắt instance đang hoạt động (instance-rs-az1).

1. Nếu instance đang hoạt động gặp sự cố hoặc không thể truy cập, BGP sẽ phát hiện lỗi trong khoảng thời gian timeout đã cấu hình.

2. VPC Route Server đánh dấu phiên BGP với instance đang hoạt động là down và rút tuyến đường ra khỏi bảng RIB.

3. Một quá trình BGP re-convergence được kích hoạt, và tuyến đường do instance dự phòng quảng bá sẽ được chọn làm đường đi tốt nhất.

4. Bảng định tuyến VPC được cập nhật để chuyển tiếp lưu lượng đến 172.16.1.1/32 qua ENI của instance dự phòng (ENI-B).

5. Lưu lượng được chuyển tiếp liền mạch sang instance dự phòng, duy trì tính sẵn sàng của ứng dụng mà không gây gián đoạn cho client.

Để kiểm tra thiết lập định tuyến, bạn có thể truy cập vào instance kiểm thử (test-instance) bằng phương pháp Systems Manager. Sau khi đăng nhập, hãy ping đến 172.16.1.1. Bạn sẽ nhận được phản hồi từ instance hiện tại đang hoạt động (instance-rs-az2).

```bash
sh-5.2$ ping 172.16.1.1
PING 172.16.1.1 (172.16.1.1) 56(84) bytes of data.
64 bytes from 172.16.1.1: icmp_seq=1 ttl=127 time=0.712 ms
64 bytes from 172.16.1.1: icmp_seq=2 ttl=127 time=0.338 ms
64 bytes from 172.16.1.1: icmp_seq=3 ttl=127 time=0.378 ms
```

{{< figurecaption src="/images/Img6-Blog2.png" caption="Hình 6. Instance2 đảm nhận vai trò hoạt động" >}}

{{< figurecaption src="/images/Img7-Blog2.png" caption="Hình 7. Bảng định tuyến đã được cập nhật để trỏ đến ENI của Inst2" >}}


**Tình huống #2: Kiểm tra lưu lượng ingress VPC** 

Hãy xem xét kịch bản khi bạn có một mô hình bảo mật tập trung, trong đó các thiết bị tường lửa (được triển khai dưới dạng EC2 instance) sẽ kiểm tra toàn bộ lưu lượng north-south hoặc east-west trong VPC của bạn. Các tường lửa này đóng vai trò then chốt trong tư thế bảo mật và phải luôn khả dụng để kiểm tra cũng như chuyển tiếp lưu lượng. Để duy trì tính sẵn sàng cao, bạn triển khai hai EC2 firewall ở hai AZ khác nhau. Mục tiêu là đảm bảo rằng nếu firewall đang hoạt động gặp sự cố, thì lưu lượng sẽ được chuyển hướng liền mạch sang firewall dự phòng.

Trong kịch bản này, chúng ta minh họa cách triển khai tính khả dụng cao (HA) và cơ chế failover cho các firewall stateful được triển khai trên nhiều AZ trong AWS, bằng cách sử dụng VPC Route Server và cập nhật định tuyến động.

Tổng quan giải pháp
Tất cả lưu lượng đi vào VPC thông qua IGW sẽ được định tuyến đến firewall để kiểm tra trước khi đi vào application subnet. Tương tự, toàn bộ lưu lượng đi ra từ application subnet cũng sẽ được định tuyến đến firewall để kiểm tra trước khi được gửi ra internet.

Hình dưới đây minh họa một thiết bị firewall được cài đặt trên một EC2 instance trong subnet A. Thiết bị này sẽ kiểm tra toàn bộ lưu lượng đi từ IGW đến subnet B (application subnet) và từ subnet B ra IGW.

{{< figurecaption src="/images/Img8-Blog2.png" caption="Hình 8. Tình huống #2 khi firewall 1 đang hoạt động" >}}

Mỗi firewall thiết lập bốn phiên BGP: hai cho subnet A và hai cho subnet B, bao gồm cả subnet ứng dụng và bảng định tuyến IGW.

Để đảm bảo chỉ có một firewall được sử dụng tại một thời điểm, độ ưu tiên đường đi trong BGP được điều chỉnh bằng cách sử dụng các metric của BGP. Chúng ta tập trung vào các metric sau:

- AS_Path: Thuộc tính BGP cho thấy chuỗi các số hiệu Hệ thống Tự trị (AS) mà một tuyến đường đã đi qua. Nó vừa đóng vai trò là cơ chế ngăn vòng lặp, vừa là tiêu chí lựa chọn đường đi, trong đó đường đi ngắn hơn sẽ được ưu tiên.

- MED (Multi-Exit Discriminator): Thuộc tính BGP được sử dụng để ảnh hưởng đến lưu lượng vào bằng cách đề xuất điểm vào ưa thích khi tồn tại nhiều kết nối giữa hai hệ thống tự trị. Giá trị MED thấp hơn sẽ được ưu tiên.

Firewall đang hoạt động sẽ quảng bá các prefix với thuộc tính BGP ưu tiên cao nhất, trong khi firewall dự phòng quảng bá cùng prefix nhưng với các thuộc tính ít ưu tiên hơn. Trong kịch bản này, chúng ta sử dụng AS path prepending, trong đó firewall dự phòng sẽ tăng độ dài AS path khi quảng bá prefix đến RSE.

**Bảng định tuyến Internet Gateway**

Các bảng định tuyến gắn với IGW (Internet Gateway) kiểm soát đường đi mà lưu lượng internet đi vào VPC sẽ sử dụng. Người dùng thường dùng nó để chèn firewall và các chức năng mạng ảo khác vào đường đi của lưu lượng internet inbound.

Cả firewall hoạt động và firewall dự phòng đều được kết nối với VPC Route Server và quảng bá CIDR của subnet ứng dụng đến các RSE. Tuy nhiên, firewall dự phòng quảng bá tuyến với AS path dài hơn. VPC Route Server sẽ chạy thuật toán chọn đường tốt nhất của BGP và cài đặt tuyến được quảng bá bởi firewall hoạt động.

Bảng định tuyến cho subnet IGW có tuyến như sau:

```bash
Application subnet CIDR ---> Active Firewall ENI
```

VPC sẽ định tuyến lưu lượng có đích đến subnet ứng dụng đến ENI của firewall hoạt động.

**Bảng định tuyến của subnet ứng dụng**
Cả tường lửa đang hoạt động lẫn dự phòng đều được kết nối với VPC Route Server và đều quảng bá địa chỉ 0.0.0.0/0 đến các RSE. Tuy nhiên, tường lửa dự phòng quảng bá 0.0.0.0/0 với một đường dẫn AS dài hơn. VPC Route Server sẽ chạy thuật toán BGP để chọn tuyến đường tốt nhất và chỉ sử dụng tuyến được quảng bá bởi tường lửa đang hoạt động.

Bảng định tuyến của subnet ứng dụng có tuyến như sau:
```bash
0.0.0.0/0 ---> ENI của tường lửa đang hoạt động
```
Điều này có nghĩa là mọi lưu lượng từ server ứng dụng sẽ đi qua tường lửa đang hoạt động trước khi ra Internet.


**Bảng định tuyến của subnet tường lửa**

Bảng định tuyến của subnet dành cho tường lửa có tuyến tĩnh sau:
```bash
0.0.0.0/0 ---> igw-id
```
Tức là tất cả lưu lượng từ đây sẽ được gửi thẳng ra Internet Gateway (IGW).

**Phát hiện chuyển đổi dự phòng với BFD**

BFD được bật trên mỗi phiên BGP giữa các tường lửa và các RSE của VPC. BFD cho phép phát hiện sự cố rất nhanh — thường dưới một giây — bằng cách liên tục trao đổi các gói điều khiển.

Trong trường hợp tường lửa gặp sự cố:

1. BFD phát hiện phiên BGP giữa tường lửa đang hoạt động và các RSE bị lỗi.

2. Các RSE đánh dấu phiên BGP này là DOWN.

3. Các RSE rút lại các tuyến ưu tiên (bao gồm cả tiền tố nội bộ và bên ngoài) mà tường lửa bị lỗi đang quảng bá.

4. Quá trình hội tụ lại BGP diễn ra — các RSE chọn tuyến thay thế (dự phòng) được quảng bá bởi tường lửa còn hoạt động.

5. Tuyến này trở thành tuyến active trong bảng điều khiển định tuyến của VPC.

6. Lưu lượng tự động được chuyển hướng sang tường lửa dự phòng.

{{< figurecaption src="/images/Img9-Blog2.png" caption="Hình 9. Tường lửa FW1 gặp sự cố dẫn đến lưu lượng được chuyển hướng sang FW2." >}}


Phục hồi/Quay lại (Failback/Recovery)
Khi tường lửa bị lỗi phục hồi và thiết lập lại các phiên BGP và BFD:

1. Nó bắt đầu quảng bá lại các thuộc tính BGP ưu tiên.

2. Các RSE phát hiện tuyến đường hấp dẫn hơn và chuyển lưu lượng trở lại tường lửa đã được phục hồi.

Quá trình này có thể được tự động hóa hoặc điều khiển theo chính sách quản trị (ví dụ: chuyển đổi dự phòng ưu tiên hoặc không ưu tiên).

Ưu điểm của việc sử dụng VPC Route Server với Failover dựa trên BGP + BFD:

1. Hội tụ nhanh: Phát hiện sự cố dưới một giây nhờ BFD.

2. Hoàn toàn tự động: Không cần script hay can thiệp thủ công.

3. Mở rộng dễ dàng: Hoạt động trên nhiều tiền tố và instance.

4. Điều khiển gốc đám mây: Tích hợp trực tiếp với lớp định tuyến của VPC.

5. Chuẩn hóa giao thức: Sử dụng hành vi BGP theo tiêu chuẩn ngành.

**Những điểm cần lưu ý**

1. Quá trình hội tụ lại định tuyến có thể gây ra một khoảng thời gian gián đoạn. Nếu ứng dụng đã tích hợp với GWLB, hãy cân nhắc sử dụng GWLB làm lựa chọn đầu tiên cho việc chuyển đổi dự phòng ứng dụng.

2. Đảm bảo tắt tính năng route propagation nếu bạn đang quản lý tuyến đường thủ công.

3. Sử dụng BFD hoặc các công cụ phát hiện sự cố nhanh khác để hội tụ nhanh hơn.

4. Đảm bảo các tuyến đường đối xứng nếu bạn thực hiện kiểm tra đường trả về (return-path inspection).

5. Bật giám sát và cảnh báo để theo dõi tình trạng hệ thống, các thay đổi về tuyến đường và các sự kiện chuyển đổi dự phòng.

**Kết luận**
Trong bài viết này, chúng ta đã tìm hiểu cách sử dụng Amazon VPC Route Server để xây dựng các thiết kế mạng trong đám mây có khả năng mở rộng, chịu lỗi và an toàn, bằng cách bật tính năng chuyển đổi dự phòng cho các ứng dụng quan trọng và triển khai kiến trúc khả dụng cao. Chúng tôi đã giới thiệu hai mô hình kiến trúc khác nhau và cung cấp các chi tiết triển khai. VPC Route Server mở ra các khả năng định tuyến nâng cao trong AWS bằng cách tích hợp các giao thức BGP và BFD theo tiêu chuẩn ngành vào mạng VPC gốc. Để bắt đầu với VPC Route Server ngay hôm nay, bạn có thể tham khảo [tài liệu](https://docs.aws.amazon.com/vpc/latest/userguide/dynamic-routing-route-server.html) và hướng dẫn  [Amazon VPC Route Server Get Started](https://docs.aws.amazon.com/vpc/latest/userguide/route-server-tutorial.html).

Cập nhật: Vào ngày 15 tháng 9 năm 2025, phiên bản trước của bài viết có sử dụng các biểu tượng kiến trúc AWS đã lỗi thời. Bài viết đã được cập nhật để phản ánh bộ biểu tượng kiến trúc AWS hiện tại.