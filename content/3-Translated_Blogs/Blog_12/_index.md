+++
title = "Blog 12"
weight =  12
chapter = false
pre = " <b> 3.12. </b>"
+++

# Hỗ trợ tùy chỉnh trên quy mô lớn: Biến một KB (Cơ sở kiến thức) Salesforce hợp nhất thành các tác tử AI tập trung vào LOB (Ngành kinh doanh)

**Bởi:** [Bhaskar Rao](https://aws.amazon.com/), [Saqib M](https://aws.amazon.com/), [Dipkumar Mehta](https://aws.amazon.com/), và [Murtuza Kainan](https://aws.amazon.com/)  

**Ngày:** 01 tháng 8 năm 2025

**Thể loại:** Nâng cao (300), Amazon AppFlow, Amazon Connect, Amazon Q, Tương tác khách hàng, Giải pháp khách hàng, Hướng dẫn kỹ thuật

---



Trong thế giới siêu kết nối ngày nay, các nhóm hỗ trợ khách hàng phải xoay xở với một danh mục sản phẩm và dịch vụ ngày càng tăng, tất cả trong một CRM duy nhất như **Salesforce**. Các nhân viên cần truy cập ngay lập tức vào thông tin đúng, nhưng họ quá thường xuyên thấy mình phải lội qua một cơ sở kiến thức nguyên khối, hợp nhất mà bao trùm mọi ngành kinh doanh (LOB) – từ thanh toán viễn thông đến bồi thường bảo hiểm đến trả hàng bán lẻ. Những giây quý giá biến thành phút khi nhân viên nhấp chuột, cuộn và lọc, và những sự chậm trễ đó chuyển trực tiếp thành các cuộc gọi dài hơn, tỷ lệ giải quyết cuộc gọi đầu tiên thấp hơn, và khách hàng bực bội.

---

## Thách thức kinh doanh & Giải pháp

Khi **Salesforce Knowledge** đồng bộ vào **Amazon Q in Connect**, theo truyền thống, nó nhập tất cả các bài viết vào một kho lưu trữ hợp nhất. Mặc dù phương pháp tiếp cận hợp nhất này hoạt động tốt cho các nhóm nhỏ hơn hoặc các doanh nghiệp một ngành, các tổ chức quản lý nhiều ngành kinh doanh (LOB) phải đối mặt với những thách thức riêng biệt ở quy mô lớn.  

Trong những môi trường phức tạp này, bạn có thể tận dụng **các tác tử AI trong Amazon Q in Connect** để tự động phân đoạn Salesforce Knowledge của bạn thành nhiều **cơ sở kiến thức cụ thể theo ngành kinh doanh (LOB)**. Mỗi LOB KB (Cơ sở kiến thức LOB) duy trì sự tích hợp Salesforce liền mạch trong khi tồn tại như một kho lưu trữ logic riêng của nó với **các câu lệnh (prompts) AI cụ thể theo lĩnh vực**.  

Phương pháp tiếp cận này mang lại các lợi ích chính bao gồm:
- Kết quả tìm kiếm siêu liên quan  
- Hiệu suất AI được tối ưu hóa  
- Thời gian giải quyết nhanh hơn  

Điều này cho phép nhân viên tập trung chính xác vào điều quan trọng cho mỗi tương tác của khách hàng thay vì lội qua nội dung không liên quan.

---

## Những gì bạn sẽ học được trong blog này

Bài viết này sẽ hướng dẫn bạn qua quy trình nâng cao cơ sở hạ tầng hỗ trợ khách hàng của bạn bằng cách sử dụng **Amazon Q in Connect** và **Salesforce Knowledge**.  

Chúng ta sẽ bắt đầu bằng cách khám phá **kiến trúc giải pháp**, chỉ cho bạn cách tích hợp liền mạch Salesforce Knowledge với Amazon Q in Connect. Bạn sẽ học cách định nghĩa nhiều **cơ sở kiến thức cụ thể theo LOB** và liên kết chúng với **các Tác tử AI tùy chỉnh và các câu lệnh AI**, điều chỉnh việc quản lý kiến thức của bạn cho từng lĩnh vực riêng biệt của doanh nghiệp.  

Sau đó, chúng ta sẽ đi sâu vào **hướng dẫn triển khai từng bước** để cấu hình trình kết nối (connector) Salesforce để đảm bảo luồng dữ liệu mượt mà. Chúng ta sẽ hướng dẫn bạn qua quy trình **tự động hóa việc nhập dữ liệu và phân loại LOB**, đảm bảo rằng kiến thức của bạn được tổ chức hiệu quả và chính xác.  

Cuối cùng, chúng ta sẽ chỉ cho bạn cách **tạo và triển khai các câu lệnh tác tử AI** cho mỗi cơ sở kiến thức, tối ưu hóa khả năng truy xuất và trình bày thông tin liên quan cho các nhân viên hỗ trợ của bạn.  

Đến cuối bài blog này, bạn sẽ có một sự hiểu biết toàn diện về cách biến đổi hệ thống quản lý kiến thức của bạn thành một công cụ mạnh mẽ, **được điều khiển bởi AI**, giúp nâng cao hiệu suất của nhân viên và sự hài lòng của khách hàng trên tất cả các ngành kinh doanh của bạn.

---

## Đo lường thành công

**Các chỉ số chính:**
- Giảm thời gian tìm kiếm của nhân viên  
- Cải thiện tỷ lệ giải quyết cuộc gọi đầu tiên  
- Sự hài lòng của khách hàng cao hơn  
- Các bảng điều khiển và thông lệ tốt nhất về báo cáo  

Đến cuối bài viết này, bạn sẽ có một kế hoạch chi tiết rõ ràng để biến đổi việc nhập Salesforce Knowledge nguyên khối của bạn thành một **khuôn khổ được phân đoạn, điều khiển bởi AI** trong **Amazon Q in Connect**, giúp trao quyền cho các nhân viên của bạn giải quyết vấn đề nhanh hơn, duy trì luồng hội thoại, và nâng cao trải nghiệm khách hàng trên mọi ngành kinh doanh.

---

## Tổng quan giải pháp

<p align="center">
  <img src="https://d2908q01vomqb2.cloudfront.net/af3e133428b9e25c55bc59fe534248e6a0c0f17b/2025/07/28/QiC-SF.drawio3.png" 
       alt="Sơ đồ kiến trúc tổng quan" 
       width="50%">
  <br>
  <em>Sơ đồ kiến trúc tổng quan</em>
</p>

# Hướng dẫn từng bước

## A: Tích hợp Salesforce với AWS AppFlow
**Dữ liệu Salesforce Knowledge** được chuyển bằng **[AWS AppFlow](https://docs.aws.amazon.com/appflow/latest/userguide/what-is-appflow.html)**.  
Các luồng riêng biệt xử lý các đơn vị kinh doanh (BU) khác nhau:
- BU Ô tô (`auto_kb`)  
- BU Tín dụng (`credit_kb`)  
- BU Thanh toán (`payments_kb`)  

AppFlow xử lý cả luồng **theo yêu cầu (on-demand)** và **luồng gia tăng (incremental)** với các bộ lọc thích hợp để chỉ lấy Kiến thức (Knowledge) liên quan đến BU cụ thể từ Salesforce, và dữ liệu được lưu trữ trong **Amazon S3** với các tiền tố tương ứng.  
- **Luồng Theo yêu cầu (On Demand Flow)**: lấy tất cả KB liên quan cho một BU cụ thể  
- **Luồng Gia tăng (Incremental Flow)**: định kỳ thăm dò (poll) Salesforce xem có bất kỳ cập nhật nào đối với Kiến thức trong Salesforce cho BU đó hay không  

---

## B: Phân tách dữ liệu trong Amazon S3
Dữ liệu từ Salesforce được lưu trữ trong một **S3 bucket** với các tiền tố cụ thể theo BU:
- `auto_kb`  
- `credit_kb`  
- `payments_kb`  

**Thông báo sự kiện S3 (S3 Event Notifications)** được cấu hình cho mỗi tiền tố, điều này sẽ kích hoạt giai đoạn tiếp theo.

---

## C: Thông báo sự kiện S3 đến Amazon SQS
Bất cứ khi nào dữ liệu mới được thêm vào S3 bucket, một **Thông báo sự kiện S3** sẽ gửi một tin nhắn đến hàng đợi **[Amazon SQS](https://aws.amazon.com/sqs/)**.  
Hàng đợi SQS đảm bảo việc **tách rời (decoupling)** và phân phối tin nhắn đáng tin cậy cho các bước xử lý sau (downstream).

---

## D: Xử lý bằng AWS Lambda
Tin nhắn SQS kích hoạt một hàm **[AWS Lambda](https://aws.amazon.com/lambda/)**, hàm này sẽ xử lý dữ liệu đến.  
- Lambda thực hiện **làm sạch (sanitization)** nội dung HTML trong mỗi bài viết kiến thức và ghi chúng dưới dạng các tệp HTML riêng biệt vào các S3 bucket cụ thể:
  - Kiến thức Ô tô (`qic-auto`)  
  - Kiến thức Tín dụng (`qic-credit`)  
  - Kiến thức Thanh toán (`qic-payments`)  

- Lambda cũng được kích hoạt khi **tìm nạp định kỳ** từ Salesforce khi các bài viết kiến thức:
  - **Được cập nhật và xuất bản** – Lambda làm sạch nội dung HTML và ghi đè tệp trong S3 bucket cụ thể  
  - **Được lưu trữ (Archived)** – Lambda xóa tệp HTML khỏi S3 bucket tương ứng  

---

## E: Các Cơ sở kiến thức đã cập nhật trong S3
Hàm Lambda xuất **dữ liệu đã được làm sạch và phân loại** vào các S3 bucket thích hợp:
- `qic-auto`  
- `qic-credit`  
- `qic-payments`  

---

## F: Tích hợp Amazon Connect
Mỗi S3 bucket được liên kết với **Cơ sở kiến thức Q in Connect (QiC KB)**:
- `qic-auto-kb`  
- `qic-credit-kb`  
- `qic-payments-kb`  

Các KB trong QiC có loại **EXTERNAL**, điều này có nghĩa là QiC tự động đồng bộ hóa với S3 bucket. Khi nội dung được thêm vào hoặc xóa khỏi S3 bucket, QiC sẽ tự động cập nhật KB của mình.  

- Các cơ sở kiến thức hỗ trợ nhân viên trong **các tương tác với khách hàng** - **Các Tác tử AI (AI Agents)** cho cả Tìm kiếm thủ công và Đề xuất câu trả lời được tạo bằng **các Câu lệnh AI (AI Prompts) tùy chỉnh** và được liên kết với trợ lý cũng như các KB – sẽ được gọi (invoked) trong quá trình tương tác với khách hàng  

**Quy trình Tương tác Khách hàng**:  
1. Khách hàng liên hệ với doanh nghiệp thông qua **Amazon Connect** 2. **Quyết định của Luồng IVR**: Xác định loại cuộc gọi/truy vấn thông qua đầu vào IVR hoặc tra cứu của bên thứ 3:
   - Ô tô  
   - Tín dụng  
   - Thanh toán  
3. Loại liên hệ được chuyển đến **AWS Lambda** để xử lý  
4. Lambda cập nhật **Phiên làm việc của Nhân viên (Agent Session)** bằng cách gọi API QiC `updateSession()` với cấu hình nhân viên dựa trên loại liên hệ:
   - Kiến thức Ô tô  
   - Kiến thức Tín dụng  
   - Kiến thức Thanh toán  

---

## G: Liên kết Trợ lý (Assistant Association)
**Cơ sở kiến thức (KB) QIC** thích hợp được liên kết với phiên làm việc bằng cách sử dụng **liên kết trợ lý**.

---

## H: Hỗ trợ Nhân viên
**Các Nhân viên Amazon Connect (Tác tử AI QIC)** truy cập cơ sở kiến thức liên quan thông qua Trợ lý QIC:
- Nhân viên bán hàng → Cơ sở kiến thức Ô tô  
- Nhân viên hỗ trợ kỹ thuật → Cơ sở kiến thức Tín dụng  
- Nhân viên nhân sự (HR) → Cơ sở kiến thức Thanh toán  

---

## Các điều kiện tiên quyết

### Môi trường AWS
- Tài khoản AWS đang hoạt động với quyền truy cập thích hợp  
- **[AWS CLI](https://aws.amazon.com/cli/)** đã được cài đặt và cấu hình  
- **[AWS CDK CLI](https://aws.amazon.com/cdk/)** đã được cài đặt  
- Quyền truy cập và các quyền hạn tại khu vực (region) mục tiêu  

### Amazon Connect
- Một instance Amazon Connect đang hoạt động  
- Hàng đợi (queue) Connect đã được cấu hình  
- Quyền truy cập quản trị viên Connect  

### Salesforce
- Một tổ chức (org) Salesforce có quyền truy cập API  
- (Đã) triển khai cơ sở kiến thức  
- Quyền truy cập Salesforce thích hợp  
- Quyền thiết lập Ứng dụng được Kết nối (Connected App) – [Salesforce Connected App](https://help.salesforce.com/articleView?id=connected_app_overview.htm)  

### AppFlow
- Trình kết nối (Connector) Salesforce cho AppFlow – [AppFlow Salesforce Connector](https://docs.aws.amazon.com/appflow/latest/userguide/salesforce.html#salesforce-setup)  

### Môi trường phát triển
- Python 3.x  
- Git  
- Trình soạn thảo mã (Code editor)  
- Kinh nghiệm phát triển AWS CDK  

### Quyền truy cập bảo mật
- Quyền IAM thích hợp  
- Quyền truy cập Salesforce thích hợp  
- Quyền truy cập Dịch vụ Amazon AppFlow


### Cấu hình & triển khai

#### 1. Clone (Sao chép) dự án
Repository URL: [https://github.com/aws-samples/sample-sf-qic-multi-lob-intgr](https://github.com/aws-samples/sample-sf-qic-multi-lob-intgr)  

```bash
    # Clone (Sao chép) kho lưu trữ
    git clone <repository-url>
    cd <project folder>
```

#### 2. Cài đặt các gói phụ thuộc (Dependencies)


```bash
pip install -r requirements.txt 
pip install -r requirements-dev.txt
```

#### 3. Cấu hình Salesforce AppFlow

##### 3.1 Tạo kết nối AppFlow
Để tạo kết nối AppFlow với Salesforce, hãy tham khảo các bước sau: [Hướng dẫn kết nối AppFlow](https://docs.aws.amazon.com/appflow/latest/userguide/flow-tutorial-connection.html)  
Để biết thêm chi tiết về các tùy chọn cấu hình khi tạo kết nối, hãy tham khảo phần: [Kết nối Amazon AppFlow với Salesforce](https://docs.aws.amazon.com/appflow/latest/userguide/salesforce.html) trong tài liệu tham khảo bổ sung này: [Trình kết nối Salesforce cho Amazon AppFlow
](https://docs.aws.amazon.com/appflow/latest/userguide/salesforce.html)

##### 3.2 Các quyền bắt buộc cho Salesforce:** Với tư cách là quản trị viên Salesforce, hãy xem lại tài liệu Salesforce thích hợp cho các mục sau:
- Quyền đọc đối tượng Knowledge  
- Quyền truy cập API đã được bật  
- Phạm vi (scopes) OAuth đã được cấu hình  

#### 4. Cấu hình AWS
Cung cấp ID tài khoản AWS, khu vực (region) và tên cho môi trường. Với mục đích của blog này, chúng ta sẽ giả định “dev” là môi trường của mình. Do đó, trong tệp `config.dev.json` trong thư mục `config/`, hãy đảm bảo rằng các giá trị sau được cập nhật cho mỗi thuộc tính:

```json
{ 
    "account": "ID-Tài-khoản-AWS-của-bạn", 
    "region": "Khu-vực-Mục-tiêu-của-bạn", 
    "env_name": "Tên-Môi-trường" //dev 
}
```

#### 5. Thiết lập Amazon Connect
Cung cấp ID instance Connect và ID hàng đợi (queue) của bạn. CDK sẽ triển khai một Luồng liên hệ (Contact Flow), và cần biết triển khai luồng này đến instance nào và bao gồm hàng đợi nào trong Luồng liên hệ.

```json
{ "connect": { "instance_id": "ID-Instance-Connect-của-bạn", "queue_id": "ID-Hàng-đợi-Connect-của-bạn" } }
```

#### 6. Ánh xạ Cơ sở Kiến thức (Knowledge Base)
Định nghĩa các **Ngành kinh doanh (LOBs) của bạn**

```json
"LOBs": [
        "LOB1",
        "LOB2",
        "LOB3"
    ]
```

Cần có thông tin sau đây để chỉ định trường định danh duy nhất (unique identifier field) của Nội dung trong Salesforce cho mỗi LOB. Ví dụ: Trong Salesforce, bạn có thể có một trường như “ProgramId__c” hoặc “LOB__c” để phân biệt các cơ sở kiến thức.


```json
"businessUnitFilters": {
  "YourLOB1": { //e.g. Credit
    "field": "Your-Classification-Field", //e.g ProgramId__c
    "value": "LOB1-Value" //e.g. Credit Dept
  }
  // Add more LOBs as defined in the "LOBs" section above
}
```

#### 7. Cấu hình AppFlow-Salesforce
Từ Bước 1 ở trên, sao chép tên kết nối đã được tạo và cập nhật cấu hình sau:

```json
{
  "connection_name": "dev-sf-connection",  // Replace with your AppFlow connection name
  "object_name": "Knowledge__kav",         
}
```


#### 8. Các trường của Bài viết Kiến thức (Knowledge Article)
Phần Trường Tùy chỉnh (Custom Field) trong đoạn mã dưới đây cần được cập nhật ở đây. Đây là các trường bổ sung mà bạn sẽ nhập từ Salesforce và cần được liệt kê trong cấu hình. **KHÔNG THAY ĐỔI Các trường Bắt buộc của Hệ thống (System Required Fields)**


```json
"projections": [
  // System Required Fields
  {"field": "Id", "data_type": "id"},
  {"field": "LastModifiedDate", "data_type": "datetime"},
  {"field": "ArticleNumber", "data_type": "string"},
  {"field": "PublishStatus", "data_type": "picklist"},  
  {"field": "UrlName", "data_type": "string"}
  
  // Custom Fields - Replace with your actual fields
  {"field": "Your-Title-Field", "data_type": "string"},
  {"field": "Your-Content-Field", "data_type": "textarea"},
  
  // Add additional fields as needed
]
```

#### 9. Bộ lọc Salesforce Knowledge
Để nguyên như sau. Nếu có các bộ lọc bổ sung bạn muốn chỉ định, hãy thêm chúng vào đây. Tài liệu tham khảo bổ sung:
- [Change Data Capture Salesforce](https://docs.aws.amazon.com/appflow/latest/userguide/flow-tutorial-salesforce-s3.html#change-data-capture-salesforce)
- [Cấu hình Salesforce AppFlow](https://docs.aws.amazon.com/appflow/latest/userguide/salesforce.html)

```json
"filters": [
  {
    "field": "PublishStatus",
    "operator": "EQUAL_TO",
    "values": ["Online", "Archived"]
  }
  // Add additional filters as needed
]
```

#### 10. Quy tắc Xác thực (Validation Rules) Salesforce Knowledge
Cập nhật thuộc tính `field` với trường Salesforce chứa nội dung của kiến thức. Trường này phải có trong danh sách Projections ở Bước 6. Cấu hình này là cần thiết để đảm bảo chúng ta không nhập nội dung trống vào QiC.

```json
"validations": [
  {
    "field": "Your-Content-Field", // Replace with your content field. Must be in the list of Projections above
    "operator": "VALIDATE_NON_NULL",
    "action": "DropRecord"
  }
  // Add additional validations as needed
]
```

**Lưu ý quan trọng:**
- `connection_name` phải khớp chính xác với tên kết nối Amazon AppFlow-Salesforce của bạn
- `object_name` phải là `Knowledge__kav`
- Đảm bảo kết nối AppFlow có các quyền thích hợp để truy cập đối tượng được chỉ định
- Kiểm tra kết nối và quyền truy cập đối tượng trước khi triển khai
- Ghi lại bất kỳ tên đối tượng tùy chỉnh hoặc tên kết nối nào đã sử dụng
- Giữ cấu hình nhất quán giữa các môi trường
- Xác thực quyền đối tượng và khả năng truy cập trường

#### 11. Bootstrap Môi trường CDK
Nếu đây là lần đầu tiên bạn sử dụng CDK trong tài khoản/khu vực này:


```bash
cdk bootstrap aws://ACCOUNT-NUMBER/REGION
```

#### 12. Xem lại (Review) CDK Diff


```bash
cdk diff
```
#### 13. Triển khai Stack

```bash
cdk deploy
```

**Các tài nguyên được tạo khi triển khai:**
- Luồng (flow) Amazon AppFlow để tích hợp Salesforce
- Các hàm Lambda để xử lý dữ liệu
- Các vai trò (roles) và chính sách (policies) IAM
- Các cơ sở kiến thức (knowledge bases) Amazon Q
- Các thành phần tích hợp Amazon Connect


### Xác minh việc triển khai

Sau khi việc triển khai hoàn tất:

- Kiểm tra bảng điều khiển (console) AWS CloudFormation để xem trạng thái của stack
- Xác minh việc tạo luồng (flow) AppFlow
- Xác nhận việc triển khai các hàm Lambda
- Kiểm tra việc thiết lập các cơ sở kiến thức Amazon Q
- Xác thực (Validate) việc tích hợp Amazon Connect

### Các bước sau triển khai

1. Liên kết các Cơ sở Kiến thức Q in Connect với Instance Connect của bạn bằng các lệnh CLI sau

- Đầu tiên, liệt kê các Cơ sở Kiến thức của bạn

```bash
aws qconnect list-knowledge-bases --region <KHU_VỰC_AWS_CỦA_BẠN>
```

- Tiếp theo, đối với mỗi Cơ sở Kiến thức, hãy tạo một Liên kết Tích hợp Connect (Connect Integration Association). Lưu ý rằng Cơ sở Kiến thức đầu tiên do CloudFormation tạo ra có thể đã được liên kết và AWS CLI có thể báo lỗi do điều này. Bạn có thể đơn giản bỏ qua các bước này cho bất kỳ Cơ sở Kiến Kến thức nào như vậy

```bash

aws connect create-integration-association --region <KHU_VỰC_AWS_CỦA_BẠN> \
--instance-id <ID_INSTANCE_CONNECT_CỦA_BẠN> \
--integration-arn <ARN_CƠ_SỞ_KIẾN_THỨC_CỦA_BẠN> \
--integration-type WISDOM_KNOWLEDGE_BASE
```

- Cuối cùng, gắn thẻ (tag) cho mỗi Cơ sở Kiến thức là `AmazonConnectEnabled=True`

```bash
aws qconnect tag-resource --region <KHU_VỰC_AWS_CỦA_BẠN> \
--resource-arn <ARN_CƠ_SỞ_KIẾN_THỨC_CỦA_BẠN> \
--tags AmazonConnectEnabled=True
```


2. Trong Bảng điều khiển (Console) Amazon Connect, điều hướng đến Amazon Q, và nhấp vào **Thêm Miền (Add Domain):**

<p align="center">
  <img src="https://d2908q01vomqb2.cloudfront.net/af3e133428b9e25c55bc59fe534248e6a0c0f17b/2025/07/28/PD-2.png" 
       alt="Thêm miền Amazon Q" 
       width="50%">
  <br>
</p>

3. Trên trang Thêm Miền (Add Domain), Chọn **Sử dụng một miền hiện có** (Use an existing domain), và chọn miền đã được tạo bởi quá trình triển khai từ danh sách thả xuống và nhấp vào **Thêm Miền** (Add Domain):
   
<p align="center">
  <img src="https://d2908q01vomqb2.cloudfront.net/af3e133428b9e25c55bc59fe534248e6a0c0f17b/2025/07/28/PD-3.png" 
       alt="Sử dụng miền hiện có" 
       width="50%">
  <br>
</p>

4. Sau khi Miền (Domain) được thêm vào, bạn sẽ thấy một trang hiển thị Miền và các Cơ sở Kiến thức (Knowledge bases) được Liên kết như hình bên dưới:

<p align="center">
  <img src="https://d2908q01vomqb2.cloudfront.net/af3e133428b9e25c55bc59fe534248e6a0c0f17b/2025/07/28/PD-4.png" 
       alt="Miền và các cơ sở kiến thức được liên kết" 
       width="50%">
  <br>
</p>

5. Chạy các Luồng AppFlow (Run the AppFlows)
- Đăng nhập vào Bảng điều khiển Quản lý AWS (AWS Management Console) và mở bảng điều khiển Amazon AppFlow tại https://console.aws.amazon.com/appflow/
- Trong khung điều hướng (navigation pane) bên trái, chọn **Luồng** (Flows). Bảng điều khiển sẽ hiển thị trang **Luồng** (Flows). Trang này chứa một bảng tóm tắt các luồng đã được tạo.
- Để khởi tạo một luồng, bạn **kích hoạt** (activate) hoặc **chạy** (run) nó. Chúng ta có 2 loại luồng được tạo: **Theo yêu cầu** (OnDemand) và **Theo lịch** (Scheduled)
- Đối với mỗi LOB, Chọn luồng **Theo yêu cầu** (OnDemand) và chọn **Xem Chi tiết** (View Details).
- Chọn **Chạy luồng** (Run flow) để chạy luồng.
<p align="center">
  <img src="https://d2908q01vomqb2.cloudfront.net/af3e133428b9e25c55bc59fe534248e6a0c0f17b/2025/07/28/PD-5.png" 
       alt="Chạy luồng AppFlow" 
       width="50%">
  <br>
</p>

- Đối với mỗi LOB, Chọn **luồng Theo lịch** (Scheduled) và chọn **Xem Chi tiết** (View Details)
- Chọn **Kích hoạt** (Activate) để kích hoạt luồng.

<p align="center">
  <img src="https://d2908q01vomqb2.cloudfront.net/af3e133428b9e25c55bc59fe534248e6a0c0f17b/2025/07/28/PD-6.png" 
       alt="Kích hoạt luồng AppFlow" 
       width="50%">
  <br>
</p>

6. Xem xét và cập nhật luồng liên hệ (contact flow) Amazon Connect
- Đăng nhập vào instance Amazon Connect của bạn.
- Dưới mục **Định tuyến** (Routing), chọn **Luồng liên hệ** (Contact Flows).
- Chọn luồng có tên: `qic-sf-contact-flow`
- Điều hướng đến Khối (Block) **Lấy thông tin đầu vào của khách hàng** (Get customer input).
- Cập nhật **Lời nhắc** (Prompts) để bao gồm các BU hoặc LOB của bạn.
- Cập nhật khối **Đặt thuộc tính liên hệ** (Set contact attributes) cho mỗi tùy chọn. Thuộc tính `LOB` là bắt buộc, và giá trị phải giống hệt như các giá trị đã cung cấp trong cấu hình CDK tại thời điểm triển khai.
<p align="center">
  <img src="https://d2908q01vomqb2.cloudfront.net/af3e133428b9e25c55bc59fe534248e6a0c0f17b/2025/07/28/PD-7.png" 
       alt="Cập nhật luồng liên hệ Connect" 
       width="50%">
  <br>
</p>
- Nhấp vào **Lưu** (Save) để lưu luồng.
- Nhấp vào **Xuất bản** (Publish) để xuất bản luồng.

7. Xác minh trạng thái luồng AppFlow
- Kích hoạt Luồng Theo yêu cầu (OnDemand Flow) trước tiên – để truy xuất nội dung kiến thức hiện có từ Salesforce
- Bắt đầu Luồng Theo lịch (Scheduled Flow) – để định kỳ thăm dò (poll) Salesforce Knowledge nhằm nhập bất kỳ nội dung bổ sung/cập nhật nào cho Salesforce Knowledge.
8. Xác minh các S3 bucket mục tiêu để kiểm tra việc đồng bộ hóa dữ liệu Salesforce
9. Giám sát (Monitor) CloudWatch logs

### Mẹo khắc phục sự cố

#### Các sự cố thường gặp và giải pháp

**Sự cố kết nối AppFlow** - Xác minh `connection_name` trong tệp cấu hình (config).  
- Kiểm tra thông tin đăng nhập (credentials) Salesforce.  
- Xác thực (Validate) token OAuth.  

**Lỗi quyền (Permission Errors)** - Xem lại các vai trò (roles) IAM.  
- Xác minh quyền truy cập API Salesforce.  

**Sự cố đồng bộ hóa cơ sở kiến thức (Knowledge Base Sync Issues)** - Xác thực cấu hình `object_name`.  
- Kiểm tra các ánh xạ trường (field mappings).  

### Dọn dẹp

Để tránh phát sinh chi phí trong tương lai, hãy xóa các tài nguyên bằng các bước sau:

- Đảm bảo hủy ánh xạ (unmap) Luồng liên hệ (Contact Flow) do quá trình triển khai tạo ra khỏi số điện thoại.  
- Trong terminal của bạn, đảm bảo bạn đang ở trong thư mục dự án.  
- Chạy lệnh: `cdk destroy`  

### Kết luận

Tóm lại, việc tích hợp Salesforce Knowledge với Amazon Q in Connect giúp tăng cường hỗ trợ nhân viên và giải quyết một số thách thức cốt lõi mà các trung tâm liên hệ phải đối mặt. Sự tích hợp này trao quyền cho các tổ chức nâng cao hoạt động của họ thông qua việc triển khai các tác tử AI cụ thể theo LOB (ngành kinh doanh), đảm bảo sự hỗ trợ theo thời gian thực, theo ngữ cảnh được thiết kế riêng cho các nhu cầu riêng biệt của các đơn vị kinh doanh khác nhau.