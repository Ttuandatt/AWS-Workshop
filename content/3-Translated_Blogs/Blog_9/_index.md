+++
title = "Blog 9"
weight =  9
chapter = false
pre = " <b> 3.9. </b>"
+++

# Amazon Braket ra mắt bộ xử lý lượng tử siêu dẫn 54-qubit IQM Emerald

**Tác giả:** Zia Mohammad, Charunethran Panchalam Govindarajan, Peter Komar, Stefan Seegerer

**Ngày:** 21/07/2025

**Chuyên mục:** Amazon Braket – Quantum Technologies

## Giới thiệu

[Amazon Braket](https://aws.amazon.com/braket/) cho phép khách hàng thiết kế và chạy thuật toán lượng tử trên nhiều loại phần cứng lượng tử thông qua một giao diện thống nhất. Hôm nay, chúng tôi mở rộng phần cứng có sẵn trên Braket bằng việc chính thức cung cấp bộ xử lý lượng tử (QPU) mới nhất của IQM. Thiết bị, có tên Emerald, là một QPU siêu dẫn 54 qubit, đem đến cho khách hàng các cổng (gates) có độ trung thực cao hơn và kết nối lưới vuông (full square lattice connectivity).

Với việc bổ sung IQM Emerald vào Braket, khách hàng hiện có thể truy cập hai bộ xử lý IQM: Garnet 20-qubit và Emerald 54-qubit mới. Cả hai thiết bị đều có sẵn qua Vùng Châu Âu (Stockholm), giúp khách hàng có nhiều lựa chọn phần cứng lượng tử phù hợp với nhu cầu nghiên cứu và yêu cầu thuật toán.

![alt text](/aws/hinhanh/hinh10.jpg)

**Hình 1: Bộ xử lý IQM Emerald**

Trong giai đoạn đầu của tính toán lượng tử, thử nghiệm trên nhiều thiết bị sẵn có là rất quan trọng để phát triển thuật toán lượng tử, nhằm mục tiêu xử lý các vấn đề phức tạp trong những lĩnh vực như tài chính, năng lượng, dược phẩm, và logistics. Khách hàng trên toàn cầu giờ có thể chạy thử nghiệm trên phần cứng lượng tử tiên tiến nhất của IQM, sử dụng truy cập theo yêu cầu (on-demand) để thiết kế và thực thi chương trình, và truy cập ưu tiên qua Hybrid Jobs để chạy các thuật toán lượng tử biến phân (variational quantum algorithms), tất cả với giá cả thanh toán theo mức sử dụng (pay-as-you-go). Với những workload yêu cầu độ trễ thấp hoặc có tính thời gian nhạy cảm, khách hàng có thể đặt trước (reserve) năng lực dành riêng trên QPU Emerald theo giờ thông qua Braket Direct.

## Kiến trúc nâng cao cho nghiên cứu lượng tử

Emerald sử dụng kiến trúc Crystal 54 của IQM, được xây dựng bằng các qubit transmon siêu dẫn sắp xếp theo lưới vuông (square lattice) và kết nối với nhau thông qua các coupler có thể điều chỉnh (tunable couplers). Cấu hình có tính kết nối cao này cho phép ánh xạ (mapping) các thuật toán lượng tử hiệu quả lên topology của thiết bị. Thiết kế lưới vuông hỗ trợ trực tiếp mã sửa lỗi bề mặt (surface-code error correction), đặt thiết bị vào vị trí phù hợp cho các ứng dụng tính toán lượng tử chịu lỗi trong tương lai.

Thiết bị hỗ trợ các phép quay X và Y bất kỳ (arbitrary X and Y rotations) như các cổng một qubit nguyên thủy (native single-qubit gates) và sử dụng cổng CZ như cổng hai-qubit nguyên thủy (native two-qubit operation). Tập cổng này cung cấp sự linh hoạt để triển khai thuật toán lượng tử trong khi vẫn duy trì hoạt động với độ trung thực cao. Dữ liệu đặc tính ban đầu cho thấy độ trung thực trung vị (median) của cổng một-qubit là 99.93% và độ trung thực trung vị của cổng hai-qubit là 99.5%. Các chỉ số hiệu năng cập nhật có thể được tìm thấy trên trang chi tiết thiết bị Emerald trong Braket Console.

![alt text](/aws/hinhanh/hinh11.jpg)

**Hình 2: Topology của QPU Emerald hiển thị lưới vuông 54-qubit với các coupler có thể điều chỉnh**

## Tính khả dụng và truy cập được cải thiện

Cả IQM Emerald và Garnet đều có mặt 19 giờ mỗi ngày, cho phép khách hàng chạy workload lượng tử theo thời gian thuận tiện bất kể múi giờ. Việc mở rộng cửa sổ khả dụng từ IQM cho phép vòng lặp phát triển và thử nghiệm liên tục cho khách hàng toàn cầu.

Khách hàng có thể truy cập các QPU đặt tại EU này trong Vùng Châu Âu (Stockholm), nghĩa là họ có thể đáp ứng các yêu cầu về cư trú dữ liệu ở châu Âu và làm việc với các khả năng hybrid quantum-classical tiên tiến trong cùng một môi trường.

## Bắt đầu với Emerald trên Braket

Braket cung cấp cho khách hàng một giao diện lập trình thống nhất, dễ sử dụng để truy cập phần cứng lượng tử. Khách hàng có thể xây dựng, kiểm thử và chạy chương trình lượng tử trên Emerald với Braket SDK, hoặc các framework phổ biến khác như Qiskit, Pennylane, và NVIDIA CUDA-Q. Để thực thi chương trình trên Emerald, chỉ cần chỉ định ARN của thiết bị trong định nghĩa device trước khi chạy mạch:
device = AwsDevice("arn:aws:braket:eu-north-1::device/qpu/iqm/Emerald")

Ví dụ Bell State (Python)
Bạn có thể chạy một mạch chỉ với vài dòng code — ví dụ tạo trạng thái Bell:
from braket.aws import AwsDevice
from braket.circuits import Circuit

device = AwsDevice("arn:aws:braket:eu-north-1::device/qpu/iqm/Emerald")

```bash
# run circuit

bell = Circuit().h(0).cnot(control=0, target=1)
result = device.run(bell, shots=1000).result()
```

(Ghi chú: đoạn code trên khởi tạo một mạch, áp cổng Hadamard (h) lên qubit 0 rồi CNOT giữa qubit 0 và 1, chạy 1000 lần (shots) và thu kết quả.)

## Khám phá vướng mắc lượng tử quy mô lớn hơn trên Emerald

Dung lượng 54-qubit của Emerald cho phép khám phá các thuật toán lượng tử cần nhiều qubit. Xây trên các [thí nghiệm trạng thái GHZ](https://aws.amazon.com/braket) đã demo trên Garnet, số qubit mở rộng và độ kết nối cao của Emerald cho phép tạo ra các trạng thái vướng mắc (entangled states) lớn hơn và các mạch lượng tử phức tạp hơn.

Dưới đây là ví dụ chuẩn bị trạng thái GHZ 49-qubit sử dụng kết nối lưới vuông của Emerald:

![alt text](/aws/hinhanh/hinh12.jpg)

(Ghi chú: ví dụ trên đặt một đường đi (path) qua topology của Emerald, áp cổng Hadamard lên qubit đầu tiên rồi nối các cặp qubit liền kề bằng CNOT để tạo GHZ, chạy 5000 shots. disable_qubit_rewiring=True ngăn Braket tự map lại qubit — tức đảm bảo mạch tuân theo topology đã chỉ định.)

## Lập trình lượng tử nâng cao với dynamic circuits

Emerald hỗ trợ khả năng [dynamic circuits](https://aws.amazon.com/blogs/quantum-computing/experiment-with-dynamic-circuits-on-iqm-garnet-with-amazon-braket/) (tính năng thử nghiệm của Amazon Braket), cho phép chương trình lượng tử đo qubit giữa chừng (mid-circuit measurement) và áp dụng phép toán lượng tử có điều kiện dựa trên kết quả đo. Khả năng này mở ra các khả năng mới cho giao thức sửa lỗi lượng tử, thuật toán thích ứng (adaptive algorithms), và tận dụng qubit hiệu quả thông qua active reset. Ví dụ sau minh họa tái sử dụng qubit bằng active reset sử dụng dynamic circuits trên Emerald:

![alt text](/aws/hinhanh/hinh13.jpg)

(Ghi chú chi tiết các lệnh trên)

- EnableExperimentalCapability() bật các khả năng thử nghiệm như dynamic circuits.

- prx(1, pi, 0) lật qubit 1 về trạng thái |1⟩ (ở đây prx là cổng xoay/đảo tương ứng).

- measure_ff(1, feedback_key=0) đo qubit 1 giữa mạch và lưu giá trị đo dưới feedback_key=0.

- cc_prx(1, pi, 0, feedback_key=0) thực hiện thao tác có điều kiện dựa trên giá trị feedback (ví dụ: nếu đo ra 1 thì làm flip lại để active reset).

- add_verbatim_box đóng gói đoạn mạch động để truyền vào device.

* Kết quả mong đợi là đếm measurement chủ yếu ra "0" do active reset hiệu quả.

## Bắt đầu ngay hôm nay

Truy cập [Amazon Braket Management Console](https://us-west-2.console.aws.amazon.com/braket/home?region=us-west-2#/dashboard) để xem cấu trúc kết nối (device topology), thời gian khả dụng của thiết bị, và cập nhật mới nhất về độ chính xác (fidelity) của cổng một qubit, hai qubit và độ chính xác phép đo (readout fidelity).

Các nhà nghiên cứu thuộc các tổ chức được công nhận có thể đăng ký nhận tín dụng AWS để hỗ trợ cho các thí nghiệm trên Amazon Braket thông qua chương trình [AWS Cloud Credits for Research](https://aws.amazon.com/government-education/research-and-technical-computing/cloud-credit-for-research/).

Để bắt đầu chạy các chương trình lượng tử của bạn trên bộ xử lý Emerald, hãy tham khảo [repository GitHub của chúng tôi để xem các notebook ví dụ và hướng dẫn](https://github.com/amazon-braket/amazon-braket-examples). Bạn có thể chạy các chương trình này bằng môi trường Jupyter notebook được quản lý (managed) của AWS hoặc từ môi trường phát triển cục bộ trên máy bạn.
