+++

title = "Blog 9"

weight = 9

chapter = false

pre = "<b>3.9.</b>"

+++

# Amazon Braket ra mắt bộ xử lý lượng tử siêu dẫn 54-qubit IQM Emerald

**Tác giả:** Zia Mohammad, Charunethran Panchalam Govindarajan, Peter Komar, Stefan Seegerer  

**Ngày:** 21/07/2025  

**Chuyên mục:** Amazon Braket -- Quantum Technologies

---

## Giới thiệu

[Amazon Braket](https://aws.amazon.com/braket/) cho phép khách hàng thiết kế và chạy thuật toán lượng tử trên nhiều loại phần cứng lượng tử thông qua một giao diện thống nhất. Hôm nay, AWS mở rộng danh mục phần cứng trên Braket bằng việc chính thức cung cấp bộ xử lý lượng tử (QPU) mới nhất của IQM -- **IQM Emerald**. Đây là một QPU siêu dẫn **54 qubit**, cung cấp độ trung thực cổng cao hơn và **kết nối lưới vuông đầy đủ (full square lattice connectivity)**.

Với việc bổ sung IQM Emerald, khách hàng hiện có thể truy cập hai bộ xử lý lượng tử của IQM trên Amazon Braket:

- **IQM Garnet** -- 20 qubit  

- **IQM Emerald** -- 54 qubit

Cả hai thiết bị đều được triển khai tại **Vùng Châu Âu (Stockholm)**, mở rộng lựa chọn phần cứng cho các hoạt động nghiên cứu và phát triển thuật toán lượng tử.

![IQM Emerald](/aws/hinhanh/hinh10.jpg)

**Hình 1:** Bộ xử lý lượng tử IQM Emerald

Trong giai đoạn hiện tại của điện toán lượng tử, việc thử nghiệm trên nhiều nền tảng phần cứng là cực kỳ quan trọng nhằm đánh giá và phát triển thuật toán cho các bài toán phức tạp trong tài chính, năng lượng, dược phẩm và logistics. Với Amazon Braket, khách hàng có thể chạy workload lượng tử:

- Theo yêu cầu (*on-demand*)

- Qua **Hybrid Jobs** cho các thuật toán biến phân

- Đặt trước tài nguyên thông qua **Braket Direct**

Tất cả đều theo mô hình **pay-as-you-go**.

---

## Kiến trúc nâng cao cho nghiên cứu lượng tử

IQM Emerald sử dụng kiến trúc **Crystal 54**, với các qubit transmon siêu dẫn được sắp xếp theo **lưới vuông (square lattice)** và kết nối bằng các **tunable coupler**. Thiết kế này cho phép ánh xạ (mapping) thuật toán lượng tử hiệu quả lên topology phần cứng.

Đáng chú ý, kiến trúc lưới vuông hỗ trợ trực tiếp **surface-code error correction**, giúp Emerald trở thành nền tảng phù hợp cho các hệ thống lượng tử chịu lỗi trong tương lai.

Thiết bị hỗ trợ:

- Các phép quay X và Y bất kỳ (arbitrary X, Y rotations) cho cổng một qubit  

- Cổng **CZ** là cổng hai qubit nguyên thủy

Dữ liệu ban đầu cho thấy:

- **Độ trung thực cổng 1-qubit (median):** 99.93%  

- **Độ trung thực cổng 2-qubit (median):** 99.5%

Các chỉ số cập nhật được hiển thị trong Braket Console.

![Emerald topology](/aws/hinhanh/hinh11.jpg)

**Hình 2:** Topology lưới vuông 54-qubit của QPU Emerald

---

## Tính khả dụng và khả năng truy cập

Cả IQM Emerald và Garnet đều khả dụng **19 giờ mỗi ngày**, tạo điều kiện thuận lợi cho khách hàng trên toàn cầu. Việc đặt thiết bị tại EU giúp đáp ứng các yêu cầu về **data residency** đồng thời tận dụng các workflow hybrid quantum--classical trong cùng môi trường AWS.

---

## Bắt đầu với IQM Emerald trên Amazon Braket

Amazon Braket cung cấp giao diện lập trình thống nhất cho nhiều framework lượng tử, bao gồm:

- Braket SDK  

- Qiskit  

- PennyLane  

- NVIDIA CUDA-Q

Để sử dụng Emerald, bạn chỉ cần chỉ định ARN của thiết bị:

```python

from braket.aws import AwsDevice

device = AwsDevice(

    "arn:aws:braket:eu-north-1::device/qpu/iqm/Emerald"

)
```

Ví dụ: Tạo trạng thái Bell

```python

from braket.circuits import Circuit

bell = Circuit().h(0).cnot(control=0, target=1)

result = device.run(bell, shots=1000).result()
```

Đoạn code trên tạo trạng thái Bell bằng cách áp dụng cổng Hadamard lên qubit 0, sau đó CNOT giữa qubit 0 và 1, chạy 1000 shots để thu kết quả.

Khám phá trạng thái vướng mắc quy mô lớn

Dung lượng 54 qubit cho phép Emerald triển khai các mạch lượng tử phức tạp hơn. Dựa trên các thí nghiệm GHZ trước đây trên Garnet, Emerald có thể tạo trạng thái GHZ 49-qubit nhờ topology kết nối cao.

Ví dụ sử dụng đường đi qua topology của Emerald, áp dụng Hadamard lên qubit đầu và chuỗi CNOT để tạo GHZ state với 5000 shots.

Lập trình nâng cao với Dynamic Circuits

IQM Emerald hỗ trợ dynamic circuits (tính năng thử nghiệm), cho phép:

Đo qubit giữa mạch (mid-circuit measurement)

Thực hiện phép toán có điều kiện dựa trên kết quả đo

Tái sử dụng qubit bằng active reset

Giải thích các thành phần chính:

EnableExperimentalCapability() -- bật dynamic circuits

measure_ff() -- đo giữa mạch

cc_prx() -- cổng lượng tử có điều kiện

add_verbatim_box() -- truyền mạch động trực tiếp xuống phần cứng

Kết quả đo chủ yếu trả về 0, chứng tỏ quá trình active reset hoạt động hiệu quả.

Bắt đầu ngay hôm nay

Bạn có thể bắt đầu với IQM Emerald bằng cách truy cập:

Amazon Braket Management Console

GitHub -- Amazon Braket Examples

AWS cũng cung cấp AWS Cloud Credits for Research cho các tổ chức nghiên cứu đủ điều kiện nhằm hỗ trợ chạy các thí nghiệm lượng tử trên Amazon Braket.