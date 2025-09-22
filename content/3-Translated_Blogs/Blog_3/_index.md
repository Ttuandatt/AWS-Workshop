+++
title = "Blog 3"
weight = 3
chapter = false
pre = " <b> 3.3. </b>"
+++

# Cách CommBank xây dựng nền tảng giao dịch CommSec có độ sẵn sàng cao và vận hành bền vững

*Kris Severijns, Aaron Bull, Henry Zhao, và Terence Lum – 19/08/2025*

---

CommSec, công ty môi giới chứng khoán trực tuyến hàng đầu tại Úc và là công ty con của Commonwealth Bank of Australia (CommBank), hỗ trợ hàng triệu khách hàng gia tăng tài sản thông qua việc đầu tư dễ dàng, tiện lợi và chi phí hợp lý vào cả thị trường trong nước lẫn quốc tế.

CommSec đóng vai trò quan trọng trong hành trình tài chính của khách hàng, cung cấp các dịch vụ thiết yếu như nghiên cứu thị trường, quản lý danh mục và thực hiện giao dịch. Với kỳ vọng từ phía khách hàng về khả năng hoạt động 24/7, nền tảng này phải duy trì độ tin cậy ở mức cao nhất. Đồng thời, là tổ chức được quản lý bởi Ủy ban Chứng khoán và Đầu tư Úc (ASIC), CommSec phải bảo đảm khả năng vận hành bền vững và tuân thủ yêu cầu về lưu trữ dữ liệu tại Úc để giữ vững sự minh bạch và an toàn cho thị trường tài chính. Trong bài viết này, chúng ta sẽ cùng tìm hiểu cách CommSec tận dụng dịch vụ AWS để xây dựng một nền tảng giao dịch vừa ổn định, vừa hiệu năng cao, vừa đáp ứng quy định khắt khe và mang đến trải nghiệm vượt trội cho khách hàng.

### Thách thức khi vận hành đa đám mây

CommSec là một trong những hệ thống trọng yếu đầu tiên của CommBank chuyển từ trung tâm dữ liệu on-premises sang môi trường public cloud. Năm 2015, CommBank bắt đầu dịch chuyển tầng web và mobile của CommSec, đến 2019 thì hoàn tất việc chuyển cả tầng ứng dụng. Là một trong những đơn vị tiên phong, CommSec ban đầu lựa chọn kiến trúc đa đám mây (active-active) để chứng minh tính bền vững của cloud, sử dụng AWS Asia Pacific (Sydney) Region như một miền lỗi (fault domain).

Tuy nhiên, việc vận hành đa đám mây mang đến nhiều khó khăn: duy trì song song hai pipeline triển khai, một mô hình vận hành trải rộng trên hai nền tảng cloud khác nhau, cùng quy trình failover phức tạp dựa vào cơ chế “witness” bên ngoài. Điều này không chỉ làm tăng chi phí vận hành mà còn kéo chậm tốc độ phát triển, hạn chế khả năng tận dụng dịch vụ native của cloud, khiến đổi mới sáng tạo bị bó hẹp.

### Hướng giải pháp

Khi AWS trở thành [nhà cung cấp cloud chiến lược của CommBank](https://www.commbank.com.au/articles/newsroom/2025/02/amazon-web-services-collaboration.html), đội ngũ CommSec đã tái kiến trúc toàn bộ tầng ứng dụng, web và mobile vào đầu năm 2025 để vận hành hoàn toàn trên AWS. Với bước chuyển đổi này, họ đã thiết lập một “ranh giới cô lập sự cố” [(fault isolation boundary)](https://docs.aws.amazon.com/whitepapers/latest/aws-fault-isolation-boundaries/abstract-and-introduction.html) mới, đơn giản hóa kiến trúc nhưng vẫn giữ được mức độ bền vững tương đương giải pháp đa đám mây trước đây.

Trong thiết kế cũ, nếu một cloud provider hoặc trung tâm dữ liệu gặp sự cố, lưu lượng sẽ được chuyển sang nền tảng còn lại. Với kiến trúc mới chỉ trên AWS, CommSec chọn [Availability Zone (AZ)](https://docs.aws.amazon.com/whitepapers/latest/aws-fault-isolation-boundaries/availability-zones.html) làm ranh giới cô lập sự cố. Nhờ Amazon [Application Recovery Controller (ARC)](https://aws.amazon.com/application-recovery-controller/) [zonal shift](https://docs.aws.amazon.com/r53recovery/latest/dg/arc-zonal-shift.html), họ có thể chuyển lưu lượng ra khỏi AZ gặp sự cố để giảm thiểu tác động đến khách hàng, đồng thời vẫn đáp ứng yêu cầu về sự tách biệt cả vật lý và logic giữa các AZ trong một Region. [ARC zonal shift](https://docs.aws.amazon.com/r53recovery/latest/dg/arc-zonal-shift.html) được tích hợp sẵn với load balancer, cho phép điều hướng lưu lượng mà không cần phụ thuộc vào [control plane](https://docs.aws.amazon.com/whitepapers/latest/aws-fault-isolation-boundaries/control-planes-and-data-planes.html). Ngoài ra, nó còn giúp quản lý tình huống “gray failure” (hệ thống bị lỗi một phần) để giảm thiểu ảnh hưởng tới khách hàng.

Việc hợp nhất hệ thống lên AWS và tận dụng ARC zonal shift để xử lý sự cố đã giúp đội ngũ CommSec đạt được nhiều lợi ích quan trọng:

- Khả năng failover tích hợp sẵn từ ARC zonal shift cho phép họ xây dựng quy trình tự động và toàn diện để nhanh chóng chuyển hướng lưu lượng ra khỏi một Availability Zone gặp sự cố.

- Playbook chi tiết được xây dựng và kiểm thử định kỳ nhằm đảm bảo các quy trình failover luôn hiệu quả và đội ngũ vận hành luôn sẵn sàng.

- Pipeline triển khai tiêu chuẩn hóa cùng cấu hình đơn giản hơn giúp việc cập nhật hệ điều hành và triển khai code nhanh gấp đôi so với trước.

- Giảm 25% dung lượng cơ bản khi vận hành nền tảng CommSec trên ba Availability Zone của AWS, so với mô hình cũ phải duy trì bốn stack trên hai cloud, từ đó cắt giảm đáng kể chi phí vận hành.

Sơ đồ sau minh họa kiến trúc giải pháp.

{{< figurecaption src="/images/Img1-Blog3.png" caption="" >}}

Nhóm CommSec đã triển khai một số cải tiến nhằm tăng cường khả năng chịu lỗi:

- Do việc scale-in và scale-out diễn ra nhiều lần mỗi ngày, quá trình mở rộng cần đảm bảo tính ổn định cao nhất. Nhóm CommSec đã loại bỏ các phụ thuộc vào tài nguyên bên ngoài trong toàn bộ quy trình bootstrap khi scale-out bằng cách lưu trữ và truy xuất các tệp nhị phân ứng dụng trực tiếp từ [Amazon Simple Storage Service (Amazon S3)](https://aws.amazon.com/s3/) trong cùng tài khoản AWS.

- Vì lưu lượng truy cập biến động mạnh, đặc biệt vào thời điểm mở cửa thị trường (lượng truy cập của CommSec thường tăng gấp ba lần chỉ trong khoảng 9:59 đến 10:02 sáng), nhóm đã thiết lập [Load Balancer Capacity Unit (LCU) reservations](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/capacity-unit-reservation.html) cho các load balancer ở tầng web. Cách này giúp đảm bảo [Application Load Balancer (ALB)](https://aws.amazon.com/elasticloadbalancing/application-load-balancer/) có đủ dung lượng ngay từ đầu phiên giao dịch, thay vì phải phụ thuộc vào cơ chế scaling phản ứng đối với các đợt tăng đột biến đã biết trước.

- Họ cũng triển khai cơ chế kiểm tra sức khỏe (health check) của ALB để phát hiện các lỗi nghiêm trọng, từ đó tự động loại bỏ các instance gặp sự cố khỏi target group. Lưu lượng truy cập sẽ được chuyển hướng khỏi các instance lỗi, đồng thời hệ thống sẽ gửi cảnh báo để đội vận hành kịp thời điều tra và xử lý.

- Ngoài ra, các kết nối [AWS Direct Connect](https://aws.amazon.com/directconnect/) mới được thiết lập từ AWS đến Australian Liquidity Centre (trung tâm đặt hạ tầng chính của Sở giao dịch chứng khoán Úc – ASX, nơi vận hành các hệ thống giao dịch, thanh toán và bù trừ) nhằm nâng cao độ tin cậy trong kết nối tới các thị trường tài chính, bao gồm cả ASX và CBOE.

### ARC zonal shift để giúp giảm thiểu sự cố

Năm 2023, AWS đã ra mắt [zonal shift](https://aws.amazon.com/about-aws/whats-new/2023/01/general-availability-amazon-route-53-application-recovery-controller-zonal-shift/), một phần của [Amazon Application Recovery Controller](https://aws.amazon.com/application-recovery-controller/). Với zonal shift, bạn có thể chuyển hướng lưu lượng ứng dụng ra khỏi một Availability Zone theo cách thức highly available cho các tài nguyên được hỗ trợ. Hành động này giúp nhanh chóng khôi phục một ứng dụng khi một Availability Zone gặp sự cố, giảm thời gian và mức độ ảnh hưởng đến ứng dụng do các sự kiện như mất điện, hỏng phần cứng hoặc phần mềm. Zonal shift hỗ trợ [Application và Network Load Balancers](https://aws.amazon.com/elasticloadbalancing/), [Amazon EC2 Auto Scaling Groups](https://aws.amazon.com/ec2/autoscaling/), và [Amazon Elastic Kubernetes Service (Amazon EKS)](https://aws.amazon.com/eks/).

Nhóm CommSec đã bật ARC zonal shift trên các ALB cho tầng web và ứng dụng của họ với [cross-zone load balancing](https://aws.amazon.com/blogs/networking-and-content-delivery/using-cross-zone-load-balancing-with-zonal-shift/) được bật. Khi bắt đầu, zonal shift thực hiện hai hành động. Thứ nhất, nó gỡ bỏ địa chỉ IP của node load balancer trong Availability Zone được chỉ định khỏi DNS, vì vậy các truy vấn mới sẽ không được phân giải đến endpoint đó. Điều này ngăn các yêu cầu mới từ client được gửi đến node đó. Thứ hai, nó chỉ thị cho các node load balancer trong các Availability Zone khác không định tuyến yêu cầu đến các target trong Availability Zone bị sự cố. Cross-zone load balancing vẫn được sử dụng trong các Availability Zone còn lại trong suốt quá trình zonal shift, như hình dưới đây.

Sau khi sự cố được giải quyết và ứng dụng hoạt động trở lại trong tất cả các Availability Zone, nhóm CommSec hủy zonal shift, và lưu lượng được phân phối lại trên cả ba Availability Zone.

{{< figurecaption src="/images/Img2-Blog3.png" caption="" >}}

### Lợi ích của ARC zonal shift

ARC zonal shift giúp các tổ chức duy trì SLA về tính khả dụng cao hơn, giảm chi phí vận hành liên quan đến các quy trình chuyển đổi dự phòng thủ công nhiều bước, và giảm thiểu thất thoát doanh thu do gián đoạn dịch vụ. Tính đơn giản của ARC zonal shift giúp các nhóm có thể thực hiện thường xuyên các bài kiểm thử quy trình di tản Availability Zone theo yêu cầu, với mức rủi ro thấp. Khả năng thực hiện xác thực định kỳ đảm bảo các quy trình failover vẫn đáng tin cậy và xây dựng sự tự tin của tổ chức trong khả năng khôi phục sau thảm họa.

*“ARC zonal shift là cách hiệu quả nhất để CommSec sử dụng các dịch vụ AWS trong khi vẫn đáp ứng yêu cầu về khả năng chịu lỗi. Nó mang lại một giải pháp sẵn có, dễ dàng hơn so với việc tự chúng tôi triển khai một giải pháp khôi phục Availability Zone. Hy vọng rằng chúng tôi sẽ không bao giờ phải dùng đến nó, nhưng việc kiểm thử định kỳ khả năng chịu lỗi đảm bảo rằng nó luôn sẵn sàng và sẽ hoạt động nếu chúng tôi cần.”*

– Henry Zhao, Kỹ sư phần mềm tại CommBank.

### Kết luận

Bằng việc sử dụng các dịch vụ AWS và triển khai kiến trúc Multi-AZ mạnh mẽ, nền tảng giao dịch của CommSec tiếp tục đáp ứng những nhu cầu khắt khe của nhà môi giới trực tuyến hàng đầu tại Úc. Sự kết hợp giữa các khả năng của ARC zonal shift, cấu hình load balancer được tối ưu hóa, cùng với các runbook và quy trình vận hành toàn diện đã giúp CommSec duy trì độ tin cậy vượt trội trong khi phục vụ hàng triệu khách hàng. Hành trình của CommSec cho thấy cách mà những quyết định kiến trúc thận trọng và các dịch vụ được quản lý bởi AWS có thể giúp các tổ chức đạt được cả vận hành xuất sắc lẫn trải nghiệm khách hàng vượt trội cho những ứng dụng tài chính quan trọng.

Để tìm hiểu thêm, hãy tham khảo AWS Fault Isolation Boundaries và Amazon Application Recovery Controller.

Link bài viết gốc: (https://aws.amazon.com/blogs/architecture/how-commbank-made-their-commsec-trading-platform-highly-available-and-operationally-resilient/)