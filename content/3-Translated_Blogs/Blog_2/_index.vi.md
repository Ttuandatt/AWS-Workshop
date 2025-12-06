+++
title = "Blog 2"
weight = 2
chapter = false
pre = " <b> 3.2. </b>"
+++

# **Tăng tốc Đổi mới Hàng không Vũ trụ: High Performance Computing (HPC) trên Amazon Web Services (AWS)**

*Tác giả: Gabe Kafity – 29/7/2025*  
*Chủ đề: Best Practices, High Performance Computing*

---

Trong ngành hàng không vũ trụ đang phát triển nhanh chóng ngày nay, khả năng đổi mới nhanh chóng và hiệu quả không chỉ là một lợi thế – mà đó là một điều cần thiết. Khi các công nghệ như UAVs (Unmanned Aerial Vehicle) tự hành, các chùm vệ tinh (satellite constellations), tên lửa tái sử dụng và thực tế tăng cường/ảo (augmented/virtual reality) tiến bộ, khả năng đổi mới nhanh chóng mang lại cho các tổ chức hàng không vũ trụ lợi thế cạnh tranh. High Performance Computing (HPC) rất quan trọng đối với đổi mới hàng không vũ trụ và đã trở thành nền tảng của sự tiến bộ trong ngành hàng không vũ trụ. Bất kể quy mô, tuổi đời hay tốc độ lặp lại (iteration speed) của một tổ chức, Amazon Web Services (AWS) luôn sẵn sàng giúp thúc đẩy các sứ mệnh hàng không vũ trụ của họ tiến lên.

Trong bài đăng này, chúng ta sẽ khám phá lý do tại sao, cách thức và những gì khách hàng hàng không vũ trụ thường làm với HPC trong AWS.


### Hiện Trạng của HPC

Cơ sở hạ tầng HPC on-premises truyền thống thường đòi hỏi đầu tư vốn đáng kể và có thể mất hàng tháng, hoặc thậm chí hàng năm, để mua sắm và triển khai. Sau khi được triển khai, các cluster thường chạy ở mức hoặc gần 100% tỷ lệ sử dụng (utilization). Tỷ lệ sử dụng cơ sở hạ tầng cao này dẫn đến thời gian chờ đợi lâu cho các HPC job mới đi vào hàng đợi (queue). Các nhà khoa học nghiên cứu và kỹ sư phải chờ đợi (thường là hàng tuần) để job của họ đi qua hàng đợi và chạy, trước khi họ có thể phân tích kết quả và lặp lại sự đổi mới của họ. Ngoài ra, chu kỳ khấu hao (depreciation cycle) của cơ sở hạ tầng HPC on-premises thường là 5-8 năm. Điều này có nghĩa là trong khi cơ sở hạ tầng HPC ngày càng tốt hơn mỗi năm, các cluster on-premises bị mắc kẹt với việc sử dụng cơ sở hạ tầng kém hiệu quả hơn cho đến khi đến lúc làm mới phần cứng (hardware refresh), lúc đó chu kỳ cơ sở hạ tầng cũ (legacy infrastructure) lại bắt đầu lại.

Ngược lại, AWS cung cấp cho các tổ chức quyền truy cập tức thì vào các tài nguyên tính toán gần như không giới hạn, cho phép họ tăng tốc đổi mới trong khi kiểm soát chi phí. Việc triển khai diễn ra chỉ trong vài phút và khách hàng chỉ trả tiền cho những gì họ sử dụng. Tận dụng các khả năng của cloud, các HPC cluster trong AWS mở rộng quy mô (scale out) để đáp ứng nhu cầu, xử lý job thành công và thu hẹp quy mô (scale back in) khi hàng đợi trống. Tính đàn hồi (elasticity) này làm giảm đáng kể thời gian chờ đợi cho các kỹ sư và nhà khoa học, trong khi chỉ phải trả tiền cho các tài nguyên khi chúng đang chạy. Ngoài ra, AWS cải thiện cơ sở hạ tầng HPC của chúng tôi với tốc độ của phần mềm. Điều này có nghĩa là thay vì chờ đợi nhiều năm để làm mới phần cứng nhằm hiện đại hóa cơ sở hạ tầng HPC, khách hàng của AWS liên tục có quyền truy cập vào cơ sở hạ tầng HPC mới nhất, hiệu suất/giá cả tốt nhất từ Amazon và các đối tác của chúng tôi (NVIDIA, Intel, AMD, v.v.).

{{< figurecaption src="/images/img1-blog2.png" caption="Hình 1: Đối lập giữa việc chạy các HPC workload on-premises (trái) so với trong AWS (phải). Bên trái bị giới hạn bởi dung lượng trung tâm dữ liệu cố định, nơi thời gian chờ đợi trong hàng đợi dài và cơ sở hạ tầng nhanh chóng trở nên lỗi thời. Bên phải có dung lượng đàn hồi (elastic capacity) có thể mở rộng theo nhu cầu, rút ngắn thời gian chờ đợi trong hàng đợi trong khi chạy trên cơ sở hạ tầng hiện đại hơn." >}}

### Các HPC Workload Chủ chốt trong Hàng không Vũ trụ

**Computational Fluid Dynamics (CFD)**

Các tổ chức hàng không vũ trụ đang tận dụng các tài nguyên tính toán mạnh mẽ của AWS để thực hiện các mô phỏng CFD phức tạp nhằm tối ưu hóa thiết kế máy bay và phân tích hệ thống đẩy (propulsion systems). Sử dụng các dịch vụ HPC của AWS, các tổ chức có thể chạy các workload như Siemens STAR-CCM+, Ansys Fluent hoặc mô phỏng OpenFOAM với hàng nghìn core, giảm thời gian mô phỏng từ hàng tuần xuống hàng giờ.

**Phân tích Cấu trúc (Structural Analysis)**

Nhu cầu của thiết kế hàng không vũ trụ hiện đại đòi hỏi phân tích cấu trúc chuyên sâu đối với những thứ như độ bền sản phẩm, độ rung và âm học (acoustics). Cho dù đó là thử nghiệm vật liệu composite mới hay thực hiện phân tích độ mỏi (fatigue analysis) trên các thành phần quan trọng, khả năng HPC của AWS cho phép khách hàng chạy nhiều mô phỏng đồng thời bằng cách sử dụng phần mềm như Dassault Systèmes Abaqus hoặc Simcenter Nastran, đẩy nhanh quá trình lặp lại thiết kế (design iteration process).


**Lập kế hoạch Sứ mệnh và Hoạt động Không gian (Mission Planning and Space Operations)**

Khi ngành hàng không vũ trụ phát triển và đổi mới, các tổ chức đang sử dụng các dịch vụ HPC của AWS để mô phỏng cơ học quỹ đạo phức tạp (orbital mechanics), tối ưu hóa việc triển khai các chùm vệ tinh (satellite constellation deployments) và quản lý các cửa sổ phóng (launch windows) một cách hiệu quả. Các mô phỏng này đòi hỏi số lượng lớn các compute cluster, cơ sở hạ tầng mạng và lưu trữ thế hệ tiếp theo, có thể dễ dàng triển khai và tự động mở rộng quy mô dựa trên nhu cầu.

{{< figurecaption src="/images/img2-blog2.png" caption="Hình 2: Ví dụ về các hình ảnh trực quan (visualizations) của các HPC workload dành cho khách hàng hàng không vũ trụ." >}}

Mỗi loại workload mô phỏng đều có các yêu cầu riêng về loại cơ sở hạ tầng mà nó chạy trên. AWS cho phép khách hàng tối ưu hóa cấu hình cơ sở hạ tầng, cluster và hàng đợi của họ để chạy hiệu quả workload mô hình hóa hoặc mô phỏng đang thực hiện.

### Bộ Công Cụ HPC của AWS
High performance computing đòi hỏi cơ sở hạ tầng hiệu quả ở mọi lớp của stack. Điều này bao gồm các công cụ tính toán (compute), lưu trữ (storage), mạng (networking) và điều phối (orchestration) cho phép các tổ chức hàng không vũ trụ đổi mới nhanh chóng. Trong phần này, chúng ta sẽ xem xét một số công cụ mà khách hàng hàng không vũ trụ sử dụng trên AWS cho các HPC workload.

**Amazon Elastic Compute Cloud (Amazon EC2)** cung cấp nền tảng tính toán rộng nhất và sâu nhất, với hơn 850 instance. Amazon EC2 có nhiều loại instance type hiệu suất cao được tối ưu hóa cho Accelerated Computing và HPC. **AWS Nitro System** được giới thiệu vào năm 2017 và được xây dựng dựa trên sự kết hợp giữa phần cứng, phần mềm và firmware được xây dựng có mục đích. Nó cung cấp cơ sở hạ tầng ảo hóa cơ bản cho các EC2 instance. Theo truyền thống, các hypervisor bảo vệ phần cứng vật lý và BIOS, ảo hóa CPU, lưu trữ, mạng và cung cấp một bộ khả năng quản lý phong phú. Với Nitro System, chúng tôi tách rời các chức năng đó, chuyển chúng sang phần cứng và phần mềm chuyên dụng, đồng thời giảm chi phí bằng cách cung cấp thực tế tất cả các tài nguyên của một server cho các instance của bạn. Điều này làm giảm thiểu chi phí ảo hóa (virtualization overhead).

{{< figurecaption src="/images/img3-blog2.png" caption="Hình 3: Nitro System làm giảm thiểu chi phí hypervisor overhead để các instance của khách hàng có thể chạy ở mức ~100% dung lượng bare metal. Vùng màu nhạt hơn cho thấy các hoạt động kỹ thuật mà Nitro đảm nhiệm, trong khi vùng màu đậm hơn cho thấy các instance của khách hàng chạy trên Nitro.">}}

Dịch vụ được quản lý (managed service) mới nhất của AWS giúp đơn giản hóa HPC trên AWS là AWS Parallel Computing Service (AWS PCS). AWS PCS giúp khách hàng dễ dàng chạy và mở rộng quy mô các HPC workload cũng như xây dựng các mô hình khoa học và kỹ thuật trên AWS bằng cách sử dụng Slurm làm trình quản lý workload. Dịch vụ được quản lý này cho phép bạn xây dựng các HPC cluster hoàn chỉnh tích hợp các tài nguyên tính toán (compute), lưu trữ (storage), mạng (networking) và hình ảnh trực quan (visualization), và mở rộng quy mô liền mạch từ 0 đến hàng nghìn instance. Thay vào đó, khách hàng có thể sử dụng AWS ParallelCluster, đây là một công cụ quản lý cluster mã nguồn mở (open-source), giàu tính năng, giúp dễ dàng cấu hình, triển khai và quản lý các HPC cluster trên AWS. Công cụ này được sử dụng thông qua các mẫu cơ sở hạ tầng dưới dạng mã (infrastructure as code templates), và có giao diện đồ họa dựa trên web tùy chọn. AWS ParallelCluster không phải là một dịch vụ được quản lý (managed service) và do đó yêu cầu khách hàng phải tự triển khai.

AWS Batch giúp bạn chạy các batch computing workload trên AWS Cloud. Batch computing là một cách phổ biến để các nhà phát triển, nhà khoa học và kỹ sư truy cập vào một lượng lớn tài nguyên tính toán. AWS Batch loại bỏ công việc nặng nhọc không tạo ra sự khác biệt trong việc cấu hình và quản lý cơ sở hạ tầng cần thiết, giống như phần mềm batch computing truyền thống. Dịch vụ này có thể cấp phát tài nguyên hiệu quả để phản hồi các job đã gửi nhằm loại bỏ các hạn chế về dung lượng (capacity constraints), giảm chi phí tính toán và cung cấp kết quả nhanh chóng.

Cho đến nay, chúng ta đã thảo luận về các tài nguyên tính toán và công cụ điều phối (orchestration tooling) cho phép các HPC workload chạy trên AWS. Có những thành phần khác quan trọng đối với cơ sở hạ tầng HPC, chẳng hạn như mạng kết nối các compute node và lưu trữ hiệu suất cao (high-performance storage). Trước tiên, hãy xem xét về mạng. 

Elastic Fabric Adapter (EFA) là một giao diện mạng cho các Amazon EC2 instance cho phép khách hàng chạy các ứng dụng đòi hỏi mức độ giao tiếp giữa các node (inter-node communications) cao trên quy mô lớn trên AWS. Giao diện phần cứng bỏ qua hệ điều hành (OS bypass hardware interface) được xây dựng tùy chỉnh của nó giúp tăng cường hiệu suất giao tiếp giữa các instance (inter-instance communications), điều này rất quan trọng để mở rộng quy mô các HPC workload có độ trễ thấp (low latency).

{{< figurecaption src="/images/img4-blog2.png" caption="Hình 4: Cho thấy network infrastructure stack của EFA, trực quan hóa cách kernel được bỏ qua để tăng tốc hiệu suất.">}}


AWS cung cấp nhiều dịch vụ lưu trữ, chẳng hạn như Amazon Simple Storage Service (Amazon S3), Amazon Elastic Block Storage (Amazon EBS), cùng nhiều dịch vụ khác. Tất cả các dịch vụ lưu trữ này có thể được sử dụng khi xây dựng các HPC cluster trong AWS. Tuy nhiên, nhiều HPC workload được hưởng lợi rất nhiều từ bộ lưu trữ chuyên biệt, chẳng hạn như Lustre – một hệ thống tệp phân tán, song song, mã nguồn mở được thiết kế cho HPC và lưu trữ dữ liệu quy mô lớn. Amazon đã giải quyết nhu cầu này cho các HPC và AI/ML workload bằng cách cung cấp Amazon FSx for Lustre.

Amazon FSx for Lustre là dịch vụ lưu trữ chia sẻ được quản lý hoàn toàn (fully managed shared storage service), được xây dựng trên hệ thống tệp song song, hiệu suất cao phổ biến nhất thế giới. Nó cho phép khách hàng tăng tốc các compute workload với bộ lưu trữ chia sẻ cung cấp độ trễ dưới mili giây (sub-millisecond latencies), thông lượng (throughput) lên đến hàng trăm GB/s, và hàng triệu IOPS, tất cả đều được quản lý hoàn toàn và có thể triển khai trong vài phút, mà không gặp khó khăn trong việc thiết lập và quản trị.

### Một Ngày của một HPC Job trên AWS

Giờ đây chúng ta đã hiểu rõ hơn về các trường hợp sử dụng HPC và các dịch vụ mà khách hàng hàng không vũ trụ đang tận dụng trên AWS, hãy tổng hợp tất cả lại thành một quy trình làm việc (workflow) chức năng. Sơ đồ dưới đây minh họa một khách hàng đang chạy các HPC workload trong môi trường hybrid cloud của họ, giữa trung tâm dữ liệu on-premises và AWS. Người dùng cuối từ bên trong ranh giới mạng của khách hàng kết nối với Login Nodes thông qua SSH. Từ Login Nodes, các HPC job được gửi đi và thêm vào hàng đợi job. Điều này kích hoạt việc phân bổ các compute node, nơi các EC2 instance được mở rộng quy mô để đáp ứng nhu cầu hàng đợi và chạy các job. Các EC2 này có khả năng kết nối với các AWS services, chạy cho đến khi HPC job hoàn thành, và sau đó tự động thu hẹp quy mô trở lại (scale back down).

{{< figurecaption src="/images/img5-blog2.png" caption="Hình 5: Ví dụ về quy trình làm việc triển khai các HPC job tận dụng AWS Parallel Computing Service.">}}


Chúng ta đã đề cập đến một số trường hợp sử dụng, dịch vụ và quy trình làm việc mà khách hàng hàng không vũ trụ tận dụng trên AWS. Bước hợp lý tiếp theo là nghe từ chính khách hàng

### Các Câu Chuyện Thành Công trong Hàng không Vũ trụ từ Thực tế

[Hypersonix Launch Systems](https://aws.amazon.com/solutions/case-studies/hypersonix-graviton-case-study/) đã giảm 92% thời gian CFD simulation pipeline của họ, từ 3 tháng xuống còn 1 tuần, bằng cách di chuyển sang AWS. Họ đã chạy các STAR-CCM+ workload on-premises, trong một HPC cluster bị sử dụng quá mức và lỗi thời. Thời gian chờ đợi trong hàng đợi kéo dài khiến các nhà nghiên cứu và kỹ sư của họ thường xuyên phải ngồi không. AWS đã trả lại thời gian cho các đội ngũ kỹ thuật này, để họ có thể đổi mới và đưa sản phẩm ra thị trường nhanh hơn. “Tôi tin rằng chúng tôi có thể nổi bật so với các công ty lớn hơn vì chúng tôi có khả năng và tài nguyên cloud mà chúng tôi cần trên AWS.”, Tiến sĩ Stephen Hall, Trưởng phòng Mô phỏng Cấu trúc Nhiệt CFD Tiên tiến tại Hypersonix Launch Systems, cho biết.

[Boom Supersonic](https://d1.awsstatic.com/events/reinvent/2021/The_future_of_HPC_is_looking_a_lot_like_ML_CMP312.pdf) sử dụng AWS để tăng tốc thiết kế và xây dựng máy bay siêu thanh của họ. Họ có thể chạy hàng nghìn mô phỏng tiên tiến đồng thời trên AWS, dẫn đến năng suất tăng ước tính gấp 6 lần so với môi trường on-prem của họ. Boom đã sử dụng hơn 53 triệu compute hours trên AWS để hoàn thành máy bay chở khách Overture của họ. “AWS, nhà cung cấp cloud hàng đầu thế giới, sẽ giúp chúng tôi liên tục tinh chỉnh các thiết kế của mình.”, Blake Scholl, Người sáng lập và CEO của Boom Supersonic, cho biết. 

Để biết thêm thông tin về các câu chuyện thành công của khách hàng, vui lòng truy cập: (https://aws.amazon.com/solutions/case-studies/)

### Kết Luận
HPC dựa trên Cloud đang cách mạng hóa cách các tổ chức hàng không vũ trụ đổi mới. AWS cung cấp khả năng mở rộng (scalability), hiệu suất và bảo mật cần thiết cho các HPC workload hàng không vũ trụ khắt khe nhất. Khi ngành công nghiệp tiếp tục phát triển, cam kết của chúng tôi trong việc hỗ trợ đổi mới hàng không vũ trụ vẫn mạnh mẽ hơn bao giờ hết.


Link bài viết gốc: (https://aws.amazon.com/blogs/hpc/accelerating-aerospace-innovation-high-performance-computing-hpc-on-amazon-web-services-aws/)