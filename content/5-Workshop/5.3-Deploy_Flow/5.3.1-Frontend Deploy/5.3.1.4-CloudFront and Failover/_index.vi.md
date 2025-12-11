+++
title = "CloudFront & Failover"
weight =  4
chapter = false
pre = " <b> 5.3.1.4.  </b>"
+++

### GIAI ĐOẠN 3: CẤU HÌNH CLOUDFRONT (CDN & FAILOVER)

### BƯỚC 3.1: Tạo Distribution

#### Step 1: Get started

-   **Distribution name:** `sgutodolist-frontend-cloudfront`

-   **Distribution type:** Giữ nguyên **Single website or app**.

-   **Domain:**

    -   Tại ô **Route 53 managed domain**, nhập: `sgutodolist.com`.

    -   (Nếu có ô Alternate domain names, nhập thêm `www.sgutodolist.com` nếu giao diện cho phép, nếu không ta sẽ thêm sau).

-   Bấm **Next**.

![](/AWS-Workshop/images/fe3.1_1.jpg)


#### Step 2: Specify origin

-   **Origin type:** Chọn **Amazon S3**.

-   **Origin:**

    -   Bấm vào ô tìm kiếm, chọn bucket Singapore: `sgutodolist-frontend-sg...`.


-   **Allow private S3 bucket access to CloudFront:**

    -   Chọn: **Allow private S3 bucket access to CloudFront - Recommended**.

-   **Cache settings:** Giữ nguyên "Use recommended cache settings...".

-   Bấm **Next**.

![](/AWS-Workshop/images/fe3.1_2.jpg)


#### Step 3: Enable security

-   **Web Application Firewall (WAF):**

    -   Chọn ô bên phải: **Do not enable security protections** (để tiết kiệm chi phí).

-   Bấm **Next**.

![](/AWS-Workshop/images/fe3.1_3.jpg)


#### Step 4: Get TLS certificate

-   **TLS certificate:**

    -   Chọn chứng chỉ ACM đã tạo: `sgutodolist.com (...)`.

-   Bấm **Next**.

![](/AWS-Workshop/images/fe3.1_4.jpg)


#### Step 5: Review and create

-   Kéo xuống dưới cùng và bấm nút màu cam **Create distribution**.

![](/AWS-Workshop/images/fe3.1_5.jpg)


* * * * *

### BƯỚC 3.1 (Bổ sung): Cấu hình sau khi tạo (Bắt buộc)

**Origin access control:** bước này làm sau khi tạo Distribution thành công, vào tab Origin của Distribution vừa tạo, click vào origin và nhấn edit

    -   Click **Create new OAC**.

    -   Name: `S3-OAC-HA`.

    -   Signing behavior: **Sign requests**.

    -   Click **Create**.

![](/AWS-Workshop/images/fe3.1_6.jpg)


Tiếp đó:

**1\. Copy Policy (Quan trọng):**

- Click vào Distribution vừa tạo

- Vào tab Origin của Distribution vừa tạo

- Click vào origin trong danh sách Origins và nhấn nút Edit

![](/AWS-Workshop/images/fe3.1_7.jpg)


-   Bấm nút **Copy policy** ở Origin access control.

![](/AWS-Workshop/images/fe3.1_8.jpg)


-   Sang tab S3 Console > Bucket Singapore > Permissions > Bucket Policy > Paste vào > Save.

![](/AWS-Workshop/images/fe3.1_9.jpg)


**2\. Thêm Default Root Object (Sửa lỗi màn hình trắng):**

-   Trong màn hình chi tiết Distribution vừa tạo, chọn tab **General** (Tab đầu tiên).

-   Kéo xuống mục **Settings**, bấm nút **Edit** (nằm bên phải mục Settings).

-   Tìm ô **Default root object**.

-   Nhập: `index.html`.

-   (Tiện thể kiểm tra mục **Alternate domain names (CNAMEs)**: Đảm bảo đã có cả `sgutodolist.com` và `www.sgutodolist.com`. Nếu thiếu thì Add item thêm vào).

-   Kéo xuống bấm **Save changes**.

![](/AWS-Workshop/images/fe3.1_10.jpg)




#### Bước 3.2: Thêm Origin Phụ (Virginia)

1.  Vào Distribution vừa tạo > Tab **Origins**.

2.  Click **Create origin**.

    -   **Origin domain:** Chọn bucket Virginia (`sgutodolist-frontend-us.s3...`).

    -   **Origin access:** Chọn lại cái `S3-OAC-HA` đã tạo lúc nãy.

    -   **Name:** `Failover-US`.

3.  Click **Create origin**.

![](/AWS-Workshop/images/fe3.2_1.jp)

![](/AWS-Workshop/images/fe3.2_2.jp)


#### Bước 3.3: Tạo Origin Group (Kích hoạt High Availability)

1.  Vẫn ở tab Origins > Click **Create origin group**.

2.  **Name:** `HighAvailability-Group`.

3.  **Origins:**

    -   Add `Primary-SG` (Lên trên - Ưu tiên 1).

    -   Add `Failover-US` (Xuống dưới - Ưu tiên 2).

4.  **Failover criteria:** Tích chọn: **500, 502, 503, 504**.

5.  Click **Create origin group**.

![](/AWS-Workshop/images/fe3.3_1.jpg)

Ta sẽ có 2 origins và 1 origin group:
![](/AWS-Workshop/images/fe3.3_2.jpg)


#### Bước 3.4: Cập nhật Behavior

1.  Tab **Behaviors** > Chọn `Default (*)` > **Edit**.

2.  **Origin and origin groups:** Đổi từ `Primary-SG` sang **`HighAvailability-Group`**.

3.  Click **Save changes**.

![](/AWS-Workshop/images/fe3.4_1.jpg)


#### Bước 3.5: Cấu hình SPA Routing (Xử lý lỗi 404 React)

1.  Tab **Error pages** > **Create custom error response**.

2.  **Rule 1 (Cho OAC):**

    -   HTTP error code: **403**.

    -   Customize error response: **Yes**.

    -   Response page path: `/index.html`.

    -   HTTP response code: **200**.

![](/AWS-Workshop/images/fe3.5_1.jpg)


3.  **Rule 2 (Cho React Router):**

    -   Tạo thêm 1 cái tương tự cho mã **404**. (Path vẫn là `/index.html`, code 200).

![](/AWS-Workshop/images/fe3.5_2.jpg)


* * * * *

<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.3-Route 53 and ACM" %}}" style="text-decoration: none; font-weight: bold;">
⬅ BƯỚC 3: Route 53 và ACM
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.5-S3 Policy" %}}" style="text-decoration: none; font-weight: bold;">
BƯỚC 5: S3 Policy ➡
</a>
</div>

