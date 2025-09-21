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