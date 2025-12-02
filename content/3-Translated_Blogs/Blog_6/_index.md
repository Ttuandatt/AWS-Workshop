+++
title = "Blog 6"
weight = 6
chapter = false
pre = " <b> 3.6. </b>"
+++

# **Kích hoạt phân tích dữ liệu Genomic và Multiomic nhanh chóng với Illumina DRAGEN™ v4.4 trên các instance Amazon EC2 F2**

_Eric Allen, Mark Azadpour, Deepthi Shankar, Olivia Choudhury, và Shyamal Mehtalia_ | 15/07/2025 | [High Performance Computing](https://aws.amazon.com/blogs/hpc/category/high-performance-computing/), [Life Sciences](https://aws.amazon.com/blogs/hpc/category/industries/life-sciences/), [Partner solutions](https://aws.amazon.com/blogs/hpc/category/post-types/partner-solutions/)

Bài viết này được đóng góp bởi Eric Allen (AWS), Olivia Choudhury (AWS), Mark Azadpour (AWS), Deepthi Shankar (Illumina), và Shyamal Mehtalia (Illumina)

![image1](../../../images/Blog8/HPCBlog-366-p1-1024x286.png)

Việc phân tích lượng dữ liệu genomic và multiomic ngày càng gia tăng đòi hỏi các giải pháp tính toán hiệu quả, có khả năng mở rộng và tiết kiệm chi phí. Amazon Web Services (AWS) tiếp tục hỗ trợ các workload này thông qua các dịch vụ tính toán tăng tốc FPGA như các [instance Amazon EC2 F2](https://aws.amazon.com/ec2/instance-types/f2/).

Giải pháp phân tích thứ cấp [DRAGEN (Dynamic Read Analysis for GENomics)](https://www.illumina.com/products/by-type/informatics-products/dragen-secondary-analysis.html) của Illumina đã khẳng định vị thế là một trong những giải pháp phân tích thứ cấp hàng đầu cho dữ liệu sequencing thế hệ mới, cung cấp các thuật toán được tối ưu hóa cao cho phân tích genomic và triển khai tăng tốc phần cứng cho các pipeline phân tích toàn diện về genomics và multiomics, bao gồm DNA germline, DNA somatic, cũng như phân tích RNA ở mức bulk và single-cell, proteomics, spatial, và nhiều hơn nữa.

DRAGEN chạy nguyên bản trên các instance EC2 F2 và cung cấp cho khách hàng một phương pháp nhanh chóng để phân tích tập dữ liệu sinh học của họ. Việc di chuyển sang F2 được đơn giản hóa vì cùng một DRAGEN AMI được sử dụng cho cả F1 và F2, và kết quả phân tích sẽ tương đương nhau. Trong bài viết này, chúng tôi sẽ thảo luận về các đặc tính hiệu năng của DRAGEN trên các môi trường tính toán AWS khác nhau, cũng như cách triển khai [phiên bản DRAGEN v4.4 mới ra mắt gần đây](https://investor.illumina.com/news/press-release-details/2025/Illumina-DRAGEN-v4-4-powers-clinical-oncology-research-and-multiomic-applications/default.aspx) trên các instance Amazon EC2 F2.

## Tổng quan về các instance Amazon EC2 F2

Các instance F2 là thế hệ thứ 2 của các instance EC2 được trang bị FPGA để tăng tốc phân tích dữ liệu genomic và multimedia trong môi trường cloud. Những instance này mang lại cải tiến đáng kể so với thế hệ trước — các instance F1 — với hiệu suất giá thành được cải thiện tốt hơn tới 60%. Dưới đây là các tính năng và thông số kỹ thuật chính của instance F2:

- Cấu hình FPGA: Instance F2 được trang bị tối đa tám AMD Virtex UltraScale+ HBM VU47P FPGAs, mỗi FPGA tích hợp 16GB bộ nhớ băng thông cao (HBM – High-Bandwidth Memory).

- Bộ xử lý: Được vận hành bởi bộ vi xử lý AMD EPYC thế hệ thứ 3 (Milan), instance F2 cung cấp tới 192 vCPU — gấp ba lần số lõi xử lý so với instance F1.

- Bộ nhớ: Instance này hỗ trợ tới 2 TiB bộ nhớ hệ thống, gấp đôi dung lượng bộ nhớ của instance F1.

- Lưu trữ: F2 đi kèm với tối đa 7.6 TiB bộ nhớ SSD NVMe, gấp đôi dung lượng lưu trữ của F1.

- Networking: Tốc độ băng thông mạng lên tới 100 Gbps, cao gấp bốn lần so với băng thông mạng có sẵn trên instance F1.

<table>
  <tr>
    <th>Instance Name</th>
    <th>FPGAs</th>
    <th>vCPUs</th>
    <th>Instance Memory</th>
    <th>NVMe Storage</th>
    <th>Network Bandwidth</th>
  </tr>
  <tr>
    <td>f1.2xlarge</td>
    <td>1</td>
    <td>8</td>
    <td>122</td>
    <td>470</td>
    <td>Up to 10 Gbps</td>
  </tr>
  <tr style="background-color:#A5CEF0;color: black;"> <!-- tô màu xanh lá -->
    <td>f1.4xlarge</td>
    <td>2</td>
    <td>16</td>
    <td>244</td>
    <td>940</td>
    <td>10 Gbps</td>
  </tr>
  <tr style="background-color:#A9D979;color: black;"> <!-- tô màu xanh lá -->
    <td>f2.6xlarge</td>
    <td>1</td>
    <td>24</td>
    <td>256</td>
    <td>950</td>
    <td>10 Gbps</td>
  </tr>
   <tr>
    <td>F2.12xlarge</td>
    <td>2</td>
    <td>48</td>
    <td>512</td>
    <td>1900</td>
    <td>25 Gbps</td>
  </tr>
   <tr>
    <td>f1.16xlarge</td>
    <td>8</td>
    <td>64</td>
    <td>976</td>
    <td>44x940</td>
    <td>25 Gbps</td>
  </tr>
   <tr>
    <td>F2.48xlarge</td>
    <td>8</td>
    <td>192</td>
    <td>2048</td>
    <td>7600</td>
    <td>100 Gbps</td>
  </tr>
</table>

*Bảng 1: Bảng so sánh thông số kỹ thuật về compute, memory, storage, và networking giữa các instance F1 và F2.*

## Phương pháp đánh giá hiệu năng

Illumina khuyến nghị sử dụng f1.4xlarge khi dùng instance F1 và f2.6xlarge khi dùng instance F2. Để đánh giá hiệu năng trên các instance này, DRAGEN đã được cấu hình và chạy trên AWS theo [hướng dẫn người dùng Illumina DRAGEN](https://help.dragen.illumina.com/product-guides/dragen-v4.4), và các liên kết tới genome reference file có thể tìm thấy trên [trang web Illumina DRAGEN Product Files](https://support.illumina.com/sequencing/sequencing_software/dragen-bio-it-platform/product_files.html).

1. Phân tích Whole Genome Sequencing (WGS) với độ phủ khoảng 35x sử dụng một mẫu có sẵn công khai. Mẫu HG002 từ [dự án NIST Genome in a Bottle](https://www.nist.gov/programs-projects/genome-bottle) đã được sử dụng. Mẫu này được phân tích bằng DRAGEN v4.4 Germline pipeline theo hai cách khác nhau. Phân tích “cơ bản” chỉ bao gồm alignment cơ bản và small variant calling, nhằm tạo điều kiện so sánh với các pipeline tin sinh học phổ biến cho mẫu germline như BWA/GATK. Phân tích “đầy đủ” sử dụng tất cả các variant caller, bao gồm cả copy number và structural variants, cùng các tùy chọn bổ sung như gọi pharmacogenetic star allele và gọi HLA (Human Leukocyte Antigen), để tạo ra một bộ genome được phân tích đầy đủ. Trong cả hai trường hợp, tham chiếu đồ thị multigenome hg38 của DRAGEN được sử dụng cho phân tích WGS. Các file dữ liệu fastq của mẫu có thể truy cập trên Amazon S3 qua các liên kết: [fastq R1](https://s3/ilmn-dragen-giab-samples/WGS/precisionFDA_v2_HG002/HG002.novaseq.pcr-free.35x.R1.fastq.gz), [fastq R2](https://s3/ilmn-dragen-giab-samples/WGS/precisionFDA_v2_HG002/HG002.novaseq.pcr-free.35x.R2.fastq.gz).

2. Phân tích Tumor-Normal sử dụng một cặp mẫu đã được khảo sát trong một [ấn phẩm về phân tích ung thư của DRAGEN](https://www.biorxiv.org/content/10.1101/2023.03.23.534011v2) trước đây. Hai mẫu này có độ phủ khoảng 110x (tumor) và 40x (normal). Các mẫu được phân tích bằng DRAGEN v4.4 Somatic pipeline, bao gồm alignment, small variant calling, cũng như phân tích CNV và SV. Phân tích Tumor-Normal sử dụng hg38 linear genome reference của DRAGEN. Các file dữ liệu fastq của mẫu có thể truy cập trên Amazon S3 qua các liên kết: [Tumor fastq R1](https://s3/dragen-wgs-tn-somatic-benchmarking-datasets/fastq/HCC1395_75pc/tumor/T_SRR7890895_75pc_1.fastq.gz), [Tumor fastq R2](https://s3/dragen-wgs-tn-somatic-benchmarking-datasets/fastq/HCC1395_75pc/tumor/T_SRR7890895_75pc_2.fastq.gz), [Normal fastq R1](https://s3/dragen-wgs-tn-somatic-benchmarking-datasets/fastq/HCC1395_75pc/normal/N_SRR7890889_1.fastq.gz), [Normal fastq R2](https://s3/dragen-wgs-tn-somatic-benchmarking-datasets/fastq/HCC1395_75pc/normal/N_SRR7890889_2.fastq.gz).

## So sánh hiệu năng về tốc độ và chi phí: Phân tích WGS

Phân tích WGS sử dụng DRAGEN v4.4 cho thấy lợi thế đáng kể về hiệu suất chi phí trên các instance Amazon EC2 F2 so với F1, đồng thời vẫn cho ra kết quả phân tích tương đương giữa hai thế hệ instance¹:

- Phân tích WGS cơ bản, bao gồm alignment và small variant calling. DRAGEN v4.4 chạy trên f2.6xlarge đạt tốc độ **nhanh hơn 1,5 lần** và **chỉ tốn 40% chi phí compute EC2** so với f1.4xlarge.

- Phân tích WGS đầy đủ, bao gồm alignment, small variant calling, calling of CNVs, SVs, repeat expansions, và variant annotation. Phân tích DRAGEN trên f2.6xlarge đạt tốc độ **nhanh gấp 2 lần** và **chỉ tốn 30% chi phí compute EC2** so với f1.4xlarge.

![image2](../../../images/Blog8/HPCBlog-366-f1.png)

*Hình 1: Instance f2.6xlarge nhanh hơn 1.5 lần trong Phân tích WGS cơ bản và nhanh hơn 2.1 lần trong Phân tích WGS đầy đủ so với f1.4xlarge.*

![image3](../../../images/Blog8/HPCBlog-366-f2.png)

*Hình 2: Chi phí compute EC2 trên f2.6xlarge chỉ bằng 40% chi phí trên f1.4xlarge cho Phân tích WGS cơ bản và bằng 30% chi phí trên f1.4xlarge cho Phân tích WGS đầy đủ.*

## So sánh hiệu năng về tốc độ và chi phí: Phân tích Tumor-Normal

Giống như kết quả WGS, trong phân tích Tumor-Normal, DRAGEN v4.4 cũng cho thấy lợi thế đáng kể về hiệu suất chi phí trên các instance Amazon EC2 F2 so với F1, đồng thời vẫn tạo ra kết quả phân tích tương đương giữa hai thế hệ instance¹:

- Phân tích Tumor-Normal, bao gồm alignment, small variant calling và calling CNVs/SVs. Phân tích bằng DRAGEN trên f2.6xlarge đạt tốc độ **nhanh hơn 1.7 lần** và **chỉ tốn 35% chi phí compute EC2** so với f1.4xlarge.

![image4](../../../images/Blog8/HPCBlog-366-f3.png)

*Hình 3: Instance f2.6xlarge nhanh hơn 1.7 lần so với f1.4xlarge trong Phân tích Tumor-Normal.*

![image5](../../../images/Blog8/HPCBlog-366-f4.png)

*Hình 4: Chi phí compute EC2 trên f2.6xlarge chỉ bằng 35% chi phí trên f1.4xlarge cho Phân tích Tumor-Normal WGS.*

## Các lợi ích khác

Các pipeline phân tích genomic truyền thống sử dụng BWA-MEM và GATK chạy trên CPU từng là tiêu chuẩn công nghiệp trong quá khứ, nhưng DRAGEN đã ngày càng được ưa chuộng nhờ những ưu thế vượt trội về tốc độ và độ chính xác. Nhiều công bố đã được bình duyệt đã so sánh tốc độ và độ chính xác của DRAGEN với các pipeline dựa trên BWA/GATK chạy trên CPU. Ví dụ, [Ziegler et al. (2022)](https://pubmed.ncbi.nlm.nih.gov/36513709/) đã phát hiện ra rằng phân tích DRAGEN trên phần cứng FPGA nhanh hơn gấp hơn 8 lần và chính xác hơn so với các pipeline dựa trên BWA/GATK chạy trên CPU, trong khi [Sedlazek et al. (2024)](https://www.nature.com/articles/s41587-024-02382-1) cũng ghi nhận DRAGEN cho hiệu suất và độ chính xác cao hơn so với các pipeline dựa trên BWA/GATK.

Việc sử dụng DRAGEN trên các instance F, được tăng tốc bởi FPGA, còn mang lại lợi thế về tiêu thụ điện năng so với các giải pháp truyền thống dựa trên CPU và GPU. FPGA vốn dĩ tiết kiệm năng lượng hơn, tiêu thụ ít điện năng hơn nhưng vẫn đạt hiệu năng tính toán tương đương cho các workload này. Điều này đặc biệt quan trọng trong các tác vụ phân tích dữ liệu genomic, nơi khối lượng dữ liệu và thời gian xử lý có thể rất lớn.

Chẳng hạn, FPGA đạt được hiệu suất trên mỗi watt cao hơn so với CPU và GPU trong các tác vụ WGS của DRAGEN. Các bộ tăng tốc dựa trên FPGA có thể cung cấp thông lượng vượt trội với mức tiêu thụ điện năng thấp hơn. Điều này là nhờ khả năng tùy chỉnh linh hoạt của FPGA, cho phép cấu hình tối ưu hóa nhằm nâng cao hiệu quả năng lượng. Ngược lại, CPU và GPU, dù mạnh mẽ, thường tiêu tốn nhiều năng lượng hơn để thực hiện cùng một tác vụ, dẫn đến chi phí vận hành cao hơn và tác động môi trường lớn hơn.

Việc tiêu thụ điện năng thấp hơn của các FPGA dẫn đến giảm các yêu cầu về làm mát, đây có thể là một yếu tố chi phí đáng kể trong các môi trường điện toán quy mô lớn. Ngoài ra, hiệu quả năng lượng của FPGA khiến chúng trở thành lựa chọn hấp dẫn cho các ứng dụng điện toán hiệu năng cao, nơi khả năng mở rộng và hiệu quả chi phí đóng vai trò then chốt.

Tóm lại, việc triển khai DRAGEN trên các instance thuộc họ ‘F’ của Amazon EC2 mang lại một giải pháp tiết kiệm năng lượng hơn cho phân tích dữ liệu genomic so với các phương pháp truyền thống dựa trên CPU hoặc GPU, đồng thời mang lại cả lợi ích về chi phí lẫn lợi ích môi trường.

## Khả năng triển khai và các tùy chọn triển khai trên cloud của instance F2

Các instance Amazon EC2 F2 hiện đã có sẵn tại nhiều Region của AWS, bao gồm US East (N. Virginia), US West (Oregon), Europe (London) và Asia Pacific (Sydney), với kế hoạch mở rộng thêm sang nhiều Region khác trong tương lai. F2 cung cấp nhiều kích cỡ khác nhau như f2.6xlarge, f2.12xlarge và f2.48xlarge, nhằm đáp ứng đa dạng nhu cầu workload. 

Khi cấu hình lưu trữ và compute để chạy các workload DRAGEN trên các instance F của AWS, bạn cần lựa chọn các tùy chọn phù hợp để cân bằng giữa hiệu năng và chi phí. Hãy cân nhắc các tùy chọn lưu trữ như là [Amazon EBS gp3 volumes](https://aws.amazon.com/ebs/general-purpose/) được cấu hình theo RAID, [Amazon FSx for Lustre](https://aws.amazon.com/fsx/lustre/) cho thông lượng cao hơn và [Amazon Elastic File System (EFS)](https://aws.amazon.com/efs/) cho lưu trữ liên tục. Ngoài ra, việc truyền trực tuyến các tệp BAM và dữ liệu tham chiếu từ [Amazon Simple Storage Service (Amazon S3)](https://aws.amazon.com/s3/) hoặc sử dụng [Mountpoint for Amazon S3](https://aws.amazon.com/s3/features/mountpoint/) để mount bucket S3 vào hệ thống file cục bộ giúp truy cập dữ liệu một cách tiết kiệm chi phí và dễ dàng. Bằng cách lựa chọn và cấu hình cẩn thận các giải pháp lưu trữ này, bạn có thể đảm bảo hiệu năng tối ưu và hiệu quả chi phí cho các workload HPC của mình. Bên cạnh đó, bạn cũng nên cân nhắc sử dụng [Illumina Connected Analytics (ICA)](https://www.illumina.com/products/by-type/informatics-products/connected-analytics.html) hoặc [AWS Batch](https://aws.amazon.com/batch/) để quản lý workflow. Illumina cung cấp hướng dẫn chi tiết về cách triển khai [DRAGEN trên AWS](https://help.dragen.illumina.com/reference/dragen-multi-cloud/dragen-on-aws) trong [tài liệu hướng dẫn người dùng DRAGEN trực tuyến](https://help.dragen.illumina.com/) của họ.

## Kết luận

Tóm lại, các instance Amazon EC2 F2 đánh dấu một bước tiến đáng kể trong lĩnh vực điện toán đám mây được cung cấp bởi FPGA, mang lại hiệu năng, bộ nhớ, dung lượng lưu trữ và khả năng kết nối mạng vượt trội so với thế hệ trước. Sự kết hợp giữa các pipeline toàn diện của DRAGEN và sức mạnh tính toán được nâng cấp của Amazon EC2 F2 cho phép xử lý nhanh hơn, hiệu quả hơn đối với các bộ dữ liệu phức tạp – từ whole genome sequencing đến phân tích single-cell RNA.

Hãy bắt đầu chuyển đổi sang F2 ngay hôm nay. Vui lòng liên hệ với đội ngũ tài khoản AWS của bạn hoặc Illumina để được hỗ trợ trong quá trình chuyển đổi sang Amazon EC2 F2 instances.

Để biết thêm thông tin về DRAGEN trên AWS, hãy tìm kiếm [DRAGEN Complete Suite trên AWS Marketplace](https://aws.amazon.com/marketplace/search/results?searchTerms=DRAGEN+Complete+Suite), xem [blog DRAGEN v4.4](https://developer.illumina.com/news-updates/dragen-v4-4), hoặc truy cập [trang chủ Illumina Informatics Solutions](https://www.illumina.com/products/by-type/informatics-products.html).

## Tài liệu tham khảo

Scheffler, K. et al. “Somatic small-variant calling methods in Illumina DRAGEN™ Secondary Analysis.” BioRxiv (2023). [https://www.biorxiv.org/content/10.1101/2023.03.23.534011v2](https://www.biorxiv.org/content/10.1101/2023.03.23.534011v2)

Sedlazeck, F. J. et al. “Comprehensive genome analysis and variant detection at scale using DRAGEN.” Nature Biotechnology (2024). [https://www.nature.com/articles/s41587-024-02382-1](https://www.nature.com/articles/s41587-024-02382-1)

Ziegler, A. et al. “Comparison of calling pipelines for whole genome sequencing: an empirical study demonstrating the importance of mapping and alignment.” Scientific Reports (2022). 
[https://pubmed.ncbi.nlm.nih.gov/36513709/](https://pubmed.ncbi.nlm.nih.gov/36513709/)

## Chú thích

Sự tương đương giữa F1 và F2 hard-filtered.vcf cùng các tệp kết quả khác dựa trên kết quả phân tích sử dụng DRAGEN 4.4.4 theo hướng dẫn sử dụng DRAGEN của Illumina. Lệnh vim diff cho thấy sự khác biệt duy nhất giữa hai tệp VCF là một mục hiển thị thời gian thực hiện phân tích. Khách hàng có thể tự thực hiện các phân tích tương tự để xác minh hoặc liên hệ với Illumina để biết thêm thông tin.

