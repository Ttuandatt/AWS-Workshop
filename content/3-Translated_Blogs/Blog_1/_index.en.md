+++
title = "Blog 1"
weight = 1
chapter = true
pre = "<b>3.1. </b>"
_showTitle = false
+++


# **Một giải pháp có khả năng mở rộng cao cho việc sao chép dữ liệu, sử dụng Amazon FSx for NetApp ONTAP và NetApp SnapMirror.**

*Tác giả: Gaurav Acharya, Jay Horne – 30/7/2025*  
*Chủ đề: Advanced (300),Amazon FSx for NetApp ONTAP, Technical How-to*

---

Những khách hàng on-premises đang sử dụng các mảng lưu trữ NetApp trong trung tâm dữ liệu của riêng họ thường áp dụng các quy tắc kiểm soát truy cập mạng và tường lửa nghiêm ngặt để bảo vệ dữ liệu của mình, tuy nhiên, kiểu bảo mật này thường đưa Network Address Translation (NAT) vào đường truyền giữa các mảng lưu trữ. ONTAP, dù được triển khai tại chỗ hay trên đám mây, đều yêu cầu các cụm lưu trữ được cấu hình với địa chỉ IP tĩnh, và giao thức SnapMirror™ được dùng để sao chép dữ liệu giữa chúng không hỗ trợ NAT. Điều này khiến việc kết nối giữa FSx for NetApp ONTAP và các phiên bản NetApp tại chỗ nằm sau tường lửa có NAT trở nên không thể. Người dùng trong những môi trường như vậy không thể dễ dàng di chuyển dữ liệu giữa hệ thống tại chỗ và Amazon FSx for NetApp ONTAP. Lý tưởng nhất, họ sẽ chọn kết nối thông qua SnapMirror qua internet công cộng, nhưng điều này là không thể trong cấu hình mặc định.

NetApp SnapMirror™ là một tính năng thường được sử dụng cho khôi phục sau thảm họa (DR), sao lưu và sao chép dữ liệu trong hệ thống lưu trữ NetApp ONTAP, cả tại chỗ lẫn trên đám mây. Amazon FSx for NetApp ONTAP bao gồm SnapMirror như một phần của dịch vụ được quản lý toàn diện trong AWS. Tuy nhiên, vì NetApp SnapMirror không hỗ trợ NAT, và các địa chỉ IP này được xác minh đối chiếu với các hệ thống tệp được ghép cặp, nên cần triển khai một thiết bị NAT dựa trên Amazon Elastic Compute Cloud (Amazon EC2) với các Elastic IPs, để đảm bảo các tiêu đề lớp 3 (L3 headers) trùng khớp giữa mỗi hệ thống tệp và internet.

Trong bài viết này, chúng tôi thảo luận về một kiến trúc và thiết kế nhằm hợp lý hóa và mở rộng quy mô cho thách thức di chuyển dữ liệu này trong môi trường AWS. Một lựa chọn khác có thể là thiết lập các đường hầm VPN riêng lẻ giữa trung tâm dữ liệu và AWS. Tuy nhiên, cách này sẽ rất khó quản lý khi mở rộng quy mô. Nguyên nhân khiến NetApp SnapMirror không hỗ trợ NAT là vì các siêu dữ liệu được trao đổi trong quá trình ghép cặp — chẳng hạn như các địa chỉ Logical Interface (LIF) — được xác minh đối chiếu với các hệ thống tệp đang được ghép cặp. Nếu chúng không khớp, kết nối sẽ thất bại. Từ thông tin này, ta có thể nói rằng NAT không phá vỡ SnapMirror, mà chính việc thay đổi địa chỉ IP mới là nguyên nhân. Vậy, nếu chúng ta có thể thực hiện NAT theo cách mà SnapMirror vẫn có thể xác minh các địa chỉ IP đó thì sao? Chúng ta chỉ cần đảm bảo rằng các tiêu đề lớp 3 (L3 Headers) trong các gói IP trùng khớp, và cách linh hoạt nhất để làm điều đó là sử dụng một lớp NAT thứ hai.

---

### Tổng quan giải pháp

Để làm cho các headers lớp 3 (L3 headers) trùng khớp, chúng ta cần một thiết bị NAT nằm giữa mỗi hệ thống tệp và internet. Thiết bị này có thể là một máy chủ Linux, tường lửa, bộ định tuyến hoặc bất kỳ thiết bị nào trong trung tâm dữ liệu có khả năng thực hiện NAT và đáp ứng đủ băng thông cần thiết. Trong bài viết này, chúng tôi triển khai một phiên bản EC2 được tối ưu hóa cho mạng dựa trên kiến trúc Graviton trong Amazon Virtual Private Cloud (Amazon VPC). Chúng tôi sử dụng phiên bản c7gn.medium, có thông lượng mạng đạt 3.5 GB/s. Kích thước phiên bản này có thể được mở rộng tùy theo nhu cầu băng thông của bạn. Vì yêu cầu về CPU và bộ nhớ là rất nhỏ, lựa chọn này mang lại hiệu suất mạng tốt nhất so với chi phí tại thời điểm viết bài.


---

### Điều kiện tiên quyết

Các điều kiện sau là cần thiết để hoàn thành giải pháp này:
- Một thiết bị NAT mà SnapMirror có thể đi qua tại mỗi hệ thống tệp (filer).
- Một subnet riêng biệt cho từng hệ thống tệp.

{{< figurecaption src="/images/Img1-Blog1.png" caption="Hình 1. Sơ đồ kiến trúc AWS minh họa hai VPC được kết nối thông qua một cổng NAT tự quản lý dựa trên Linux." >}}



### Ví dụ

Cấu hình ví dụ được triển khai từ AWS đến AWS để đảm bảo tính trực quan, như minh họa trong hình trên, nhưng một trong hai phía đều có thể được thay thế bằng bất kỳ thiết bị NAT nào thực hiện chức năng tương tự. Tương tự, ví dụ này dựa trên hệ thống tệp FSx for ONTAP trong một Single-[Availability Zone (AZ)](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/). Nếu bạn đang sử dụng hệ thống tệp Multi-AZ, chúng tôi khuyến nghị triển khai hai phiên bản Amazon EC2 trong từng Availability Zone (AZ) của hệ thống tệp Multi-AZ và định tuyến lưu lượng thông qua phiên bản nằm trong từng AZ đó.

### EIPs

Trong ví dụ của chúng tôi, chúng tôi sử dụng một địa chỉ EIP cho mỗi giao diện liên cụm (inter-cluster interface) của FSx for ONTAP. Ban đầu, chúng tôi sẽ yêu cầu bốn địa chỉ EIP chưa được gán cho bất kỳ tài nguyên nào. Việc phân bổ cổng có thể cho phép sử dụng ít địa chỉ IP hơn, nhưng điều đó nằm ngoài phạm vi của ví dụ này.

### Security Group

Việc gán trực tiếp các địa chỉ EIP cho Amazon EC2 mà không có bất kỳ giới hạn nào là một thực hành bảo mật kém. Do đó, chúng tôi đã tạo một nhóm bảo mật (security group) trong mỗi VPC và cho phép toàn bộ lưu lượng đến từ bốn địa chỉ EIP này. Về mặt kỹ thuật, chỉ cần mở các cổng TCP 10000, 11104, 11105 và ICMP là đủ, nhưng bộ định tuyến của chúng tôi chỉ chuyển tiếp các cổng này.
Phần sau đây tóm tắt cấu hình mạng mẫu cho hai triển khai FSx for ONTAP. Các địa chỉ IP được liệt kê chỉ nhằm mục đích minh họa — các giá trị thực tế trong môi trường của bạn sẽ khác tùy theo cấu hình mạng.
Side A:
VPC: 10.1.0.0/16FSx ONTAP inter-cluster endpoint 1: 10.1.0.137FSx ONTAP inter-cluster endpoint 2: 10.1.0.125inter_1 EIP: 18.190.143.162inter_2 EIP: 3.128.12.212
Side B:
VPC: 10.2.0.0/16FSx ONTAP inter-cluster endpoint 1: 10.2.0.155FSx ONTAP inter-cluster endpoint 2: 10.2.0.110inter_1 EIP: 3.135.134.67inter_2 EIP: 3.146.166.253

### Amazon EC2

Để xử lý các NAT, hãy triển khai một phiên bản EC2 chạy RedHat 9, kèm theo nhóm bảo mật mà chúng ta đã tạo trước đó. RedHat không phải là bắt buộc, và bất kỳ bản phân phối Linux nào hỗ trợ [nftables](https://www.redhat.com/en/blog/using-nftables-red-hat-enterprise-linux-8) đều có thể hoạt động cho bài thực hành này. Đối với mỗi phiên bản EC2, chúng ta cần gắn kết hai địa chỉ EIP trong số các địa chỉ đã tạo. Mỗi địa chỉ EIP này phải được liên kết với một địa chỉ IP riêng (private IP) khác nhau. Cuối cùng, chúng ta phải tắt kiểm tra nguồn/đích (source/destination check) trên giao diện mạng. Điều này cho phép Amazon EC2 gửi các gói tin có địa chỉ IP nguồn không thuộc quyền sở hữu của nó.

{{< figurecaption src="/images/Img2-Blog1.png" caption="Hình 2. Ảnh chụp màn hình của trang tổng quan mạng (network summary page) của một phiên bản EC2, trong đó địa chỉ IP riêng (private IP) và địa chỉ IP công cộng (public IP) được tô sáng (highlighted)." >}}

### nftables

Trên mỗi phiên bản Linux này, chúng ta cần thêm một số quy tắc nftables để xử lý các kết nối. Điều này tạo ra một ánh xạ 1:1 giữa các giao diện của cụm FSx for ONTAP và một địa chỉ EIP. Đối với môi trường ví dụ của chúng ta, cấu hình nftables cho Side B sẽ như sau.

### Tùy chọn 1: Chỉnh sửa trực tiếp tệp nftables.

```
table ip nat {
   chain prerouting {
       type nat hook prerouting priority dstnat; policy accept;
       tcp dport 11104 ip daddr 10.1.0.135 dnat to 10.1.0.125
       tcp dport 11105 ip daddr 10.1.0.135 dnat to 10.1.0.125
       tcp dport 10000 ip daddr 10.1.0.135 dnat to 10.1.0.125
       icmp type { echo-reply, echo-request } ip daddr 10.1.0.135 dnat to 10.1.0.125
       tcp dport 11104 ip daddr 10.1.0.123 dnat to 10.1.0.137
       tcp dport 11105 ip daddr 10.1.0.123 dnat to 10.1.0.137
       tcp dport 10000 ip daddr 10.1.0.123 dnat to 10.1.0.137
       icmp type { echo-reply, echo-request } ip daddr 10.1.0.123 dnat to 10.1.0.137
       ip daddr 10.2.0.110 dnat to 3.146.166.253
       icmp type { echo-reply, echo-request } ip daddr 10.2.0.110 dnat to 3.146.166.253
       ip daddr 10.2.0.155 dnat to 3.135.134.67
       icmp type { echo-reply, echo-request } ip daddr 10.2.0.155 dnat to 3.135.134.67
   }

   chain postrouting {
       type nat hook postrouting priority srcnat; policy accept;
       ip saddr 10.1.0.125 snat to 10.1.0.135
       icmp type { echo-reply, echo-request } ip saddr 10.1.0.125 snat to 10.1.0.135
       ip saddr 10.1.0.137 snat to 10.1.0.123
       icmp type { echo-reply, echo-request } ip saddr 10.1.0.137 snat to 10.1.0.123
       tcp dport 11104 ip saddr 3.146.166.253 snat to 10.2.0.110
       tcp dport 11105 ip saddr 3.146.166.253 snat to 10.2.0.110
       tcp dport 10000 ip saddr 3.146.166.253 snat to 10.2.0.110
       icmp type { echo-reply, echo-request } ip saddr 3.146.166.253 snat to 10.2.0.110
       tcp dport 11104 ip saddr 3.135.134.67 snat to 10.2.0.155
       tcp dport 11105 ip saddr 3.135.134.67 snat to 10.2.0.155
       tcp dport 10000 ip saddr 3.135.134.67 snat to 10.2.0.155
       icmp type { echo-reply, echo-request } ip saddr 3.135.134.67 snat to 10.2.0.155
   }
}
```

### Tùy chọn 2: Script cấu hình nftables CLI

```
#!/bin/bash
# Install nftables and enable ip forwarding in the kernel
dnf install -y
echo 1 > /proc/sys/net/ipv4/ip_forward
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf

# Create the pre-routing and postrouting chains in nftables.
nft add table ip nat
nft -- add chain ip nat prerouting { type nat hook prerouting priority -100 \; }
nft add chain ip nat postrouting { type nat hook postrouting priority 100 \; }

# Unmap any incoming packets from the internet to the local fsx interface
# Map packets destined to 3.146.166.253(10.2.0.70) -> 10.2.0.110
nft add rule ip nat prerouting tcp dport 11104 ip daddr 10.2.0.70 dnat to 10.2.0.110
nft add rule ip nat prerouting tcp dport 11105 ip daddr 10.2.0.70 dnat to 10.2.0.110
nft add rule ip nat prerouting tcp dport 10000 ip daddr 10.2.0.70 dnat to 10.2.0.110
nft add rule ip nat prerouting icmp type { echo-request, echo-reply } ip daddr 10.2.0.70 dnat to 10.2.0.110
# Map packets destined to 3.135.134.67(10.2.0.186) -> 10.2.0.155
nft add rule ip nat prerouting tcp dport 11104 ip daddr 10.2.0.186 dnat to 10.2.0.155
nft add rule ip nat prerouting tcp dport 11105 ip daddr 10.2.0.186 dnat to 10.2.0.155
nft add rule ip nat prerouting tcp dport 10000 ip daddr 10.2.0.186 dnat to 10.2.0.155
nft add rule ip nat prerouting icmp type { echo-request, echo-reply } ip daddr 10.2.0.186 dnat to 10.2.0.155

# Map any outgoing packets from the local fsx interface to its respective public IP
# 10.2.0.110 -> 3.146.166.253(10.2.0.70)
nft add rule ip nat postrouting ip saddr 10.2.0.110 snat to 10.2.0.70
nft add rule ip nat postrouting icmp type { echo-request, echo-reply } ip saddr 10.2.0.110 snat to 10.2.0.70
# 10.2.0.155 -> 3.135.134.67(10.2.0.186)
nft add rule ip nat postrouting ip saddr 10.2.0.155 snat to 10.2.0.186
nft add rule ip nat postrouting icmp type { echo-request, echo-reply } ip saddr 10.2.0.155 snat to 10.2.0.186

# Unmap any incoming packets for the remote EIPs to the originating FSX internal IP
# 3.128.12.212 -> 10.1.0.125
nft add rule ip nat postrouting tcp dport 11104 ip saddr 3.128.12.212 snat to 10.1.0.125
nft add rule ip nat postrouting tcp dport 11105 ip saddr 3.128.12.212 snat to 10.1.0.125
nft add rule ip nat postrouting tcp dport 10000 ip saddr 3.128.12.212 snat to 10.1.0.125
nft add rule ip nat postrouting icmp type { echo-request, echo-reply } ip saddr 3.128.12.212 snat to 10.1.0.125
# 18.190.143.162 -> 10.1.0.137
nft add rule ip nat postrouting tcp dport 11104 ip saddr 18.190.143.162 snat to 10.1.0.137
nft add rule ip nat postrouting tcp dport 11105 ip saddr 18.190.143.162 snat to 10.1.0.137
nft add rule ip nat postrouting tcp dport 10000 ip saddr 18.190.143.162 snat to 10.1.0.137
nft add rule ip nat postrouting icmp type { echo-request, echo-reply } ip saddr 18.190.143.162 snat to 10.1.0.137

# Map any outgoing packets destined to a remote fsx interface to their respective public IP
# 10.1.0.125 -> 3.128.12.212
nft add rule ip nat prerouting ip daddr 10.1.0.125 dnat to 3.128.12.212
nft add rule ip nat prerouting icmp type { echo-request, echo-reply } ip daddr 10.1.0.125 dnat to 3.128.12.212
# 10.1.0.137 -> 18.190.143.162
nft add rule ip nat prerouting ip daddr 10.1.0.137 dnat to 18.190.143.162
nft add rule ip nat prerouting icmp type { echo-request, echo-reply } ip daddr 10.1.0.137 dnat to 18.190.143.162

# Persist the config
nft list ruleset > /etc/sysconfig/nftables.conf
```

### Bảng định tuyến

Khi cả hai bộ định tuyến đã được cấu hình, chúng ta cần đảm bảo rằng lưu lượng SnapMirror sẽ đi qua chúng. Để thực hiện điều này, chúng ta cập nhật bảng định tuyến (route table) được liên kết với FSx for ONTAP để gửi lưu lượng từ VPC ở xa đến giao diện mạng (network interface) của các phiên bản EC2. Ví dụ, ở Side B, chúng ta thêm một tuyến (route) trỏ 10.1.0.0/16 đến Elastic Network Interface của phiên bản EC2. Ở Side A, chúng ta sẽ làm ngược lại: trỏ 10.2.0.0/16 đến EC2 instance tương ứng.

{{< figurecaption src="/images/Img3-Blog1.png" caption="Hình 3. Bảng định tuyến (Route table) được liên kết với FSx for ONTAP." >}}

### FSx cho ONTAP security group

Là bước thiết lập cuối cùng, chúng ta cần cho phép mạng VPC từ xa được kết nối đến các giao diện của FSx for ONTAP. Để làm điều này, chúng ta đã thêm dải địa chỉ 10.0.0.0/8 vào nhóm bảo mật (security group) trên cả hai phiên bản FSx for ONTAP.

### Kết nối ngang hàng giữa các hệ thống tệp

Khi các kết nối mạng đã được thiết lập, việc còn lại là [kết nối ngang hàng (peer) giữa các hệ thống tệp FSx for ONTAP](https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/migrating-fsx-ontap-snapmirror.html#cluster-peering). Trước tiên, chúng ta đăng nhập vào Side A và bắt đầu yêu cầu peering.

{{< figurecaption src="/images/Img4-Blog1.png" caption="" >}}

Sau đó, chúng ta đăng nhập vào Side B và thực thi cùng một lệnh, nhưng không sử dụng tùy chọn generate passphrase và dùng các địa chỉ IP từ Side A. Thao tác này được thực hiện từ phía Side B của hệ thống tệp FSx for ONTAP.

{{< figurecaption src="/images/Img5-Blog1.png" caption="" >}}


Từ đây, các [SVM (Storage Virtual Machine) có thể được kết nối ngang hàng (peered)](https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/migrating-fsx-ontap-snapmirror.html#svm-peering) và [một mối quan hệ SnapMirror có thể được tạo ra](https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/migrating-fsx-ontap-snapmirror.html#snapmirror-relationship).


### Dọn dẹp

Việc chạy các phiên bản EC2 và hệ thống tệp FSx for ONTAP sẽ phát sinh chi phí. Hãy nhớ xóa và chấm dứt (terminate) các tài nguyên này nếu chúng không còn cần thiết. Để xóa một hệ thống tệp, hãy làm theo hướng dẫn trong tài liệu [hướng dẫn người dùng FSx for NetApp ONTAP](https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/getting-started.html#getting-started-step3). Để [chấm dứt các phiên bản EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/terminating-instances.html), hãy truy cập phần Terminate Your Instance trong tài liệu hướng dẫn người dùng Amazon EC2.


Link bài viết gốc: (https://aws.amazon.com/blogs/storage/highly-scalable-solution-design-to-replicate-data-using-amazon-fsx-for-netapp-ontap-and-snapmirror/)