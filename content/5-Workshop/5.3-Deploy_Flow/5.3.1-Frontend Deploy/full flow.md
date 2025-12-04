**Mô hình kiến trúc:**

1.  **Domain:** Route 53 (DNS) + ACM (SSL Certificate).

2.  **CDN:** CloudFront (Global Edge Network).

3.  **Storage (Primary):** S3 Singapore (`ap-southeast-1`).

4.  **Storage (Failover):** S3 N. Virginia (`us-east-1`).

5.  **Replication:** Tự động copy code từ Sing -> Virginia.

6.  **Security:** OAC (Origin Access Control) - Private Bucket.

* * * * *

### GIAI ĐOẠN 1: CHUẨN BỊ STORAGE (S3 & REPLICATION)

Tạo nơi chứa resources, và một cơ chế tự động đồng bộ.

#### Bước 1.1: Tạo Bucket Chính (Singapore)

1.  Vào **S3 Console** > Chọn Region **Asia Pacific (Singapore)**.

2.  Click **Create bucket**.

    -   **Bucket name:** `sgutodolist-frontend-sg`.

    -   **Object Ownership:** ACLs disabled (Recommended).

    -   **Block Public Access:** ✅ **Block all public access** (Chúng ta dùng OAC nên bucket phải kín).

    -   **Bucket Versioning:** ✅ **Enable** (Bắt buộc để chạy Replication).

    -   **Default encryption:** Server-side encryption with Amazon S3 managed keys (SSE-S3).

3.  Click **Create bucket**.




#### Bước 1.2: Tạo Bucket Phụ (Virginia)

1.  Đổi Region sang **US East (N. Virginia)**.

2.  Click **Create bucket**.

    -   **Bucket name:** `sgutodolist-frontend-us`.

    -   **Block Public Access:** ✅ **Block all public access**.

    -   **Bucket Versioning:** ✅ **Enable**.

3.  Click **Create bucket**.

#### Bước 1.3: Cấu hình Replication (Tự động Sync)

1.  Quay lại bucket **Singapore** (`sgutodolist-frontend-sg`).

2.  Tab **Management** > Mục **Replication rules** > Click **Create replication rule**.

    -   **Rule name:** `SyncToUS`.

    -   **Status:** Enabled.

    -   **Source bucket:** Apply to all objects in the bucket.

    -   **Destination:** Choose a bucket in this account > Chọn `sgutodolist-frontend-us` (nhớ chọn region us-east-1 để tìm thấy).

    -   **IAM Role:** Chọn **Create new role** (AWS tự tạo quyền cho bạn).

3.  Click **Save**. Khi hỏi "Replicate existing objects?", chọn **No** (vì bucket đang rỗng).

* * * * *

### GIAI ĐOẠN 2: CHUẨN BỊ TÊN MIỀN & BẢO MẬT (ROUTE 53 & ACM)

Trước khi tạo CloudFront, ta cần chuẩn bị Chứng chỉ bảo mật (SSL) và DNS.

#### Bước 2.1: Tạo Hosted Zone (Nếu chưa có)

1.  Vào **Route 53** > **Hosted zones**.

2.  Nếu chưa có domain `sgutodolist.com` trong list:

    -   Click **Create hosted zone**.

    -   Domain name: `sgutodolist.com`.

    -   Type: Public hosted zone.

    -   Click **Create**. (Nhớ cập nhật Nameservers bên nhà cung cấp tên miền nếu bạn mua domain ở ngoài AWS).

#### Bước 2.2: Xin cấp chứng chỉ SSL (ACM) - Quan trọng!

⚠️ **LƯU Ý:** Chứng chỉ cho CloudFront **BẮT BUỘC** phải nằm ở region **US East (N. Virginia)**.

1.  Chuyển Region console về **US East (N. Virginia)**.

2.  Vào **AWS Certificate Manager (ACM)** > **Request certificate**.

3.  Chọn **Request a public certificate** > Next.

4.  **Domain names:**

    -   `sgutodolist.com`

    -   `*.sgutodolist.com` (Để dùng cho cả www).

5.  **Validation method:** DNS validation (Recommended).

6.  Click **Request**.

7.  Trong danh sách Certificates, bấm vào ID vừa tạo (Status: Pending validation).

8.  Mục **Domains**, bấm nút **Create records in Route 53**.

9.  Click **Create records**.

10. Đợi vài phút đến khi Status chuyển sang **Issued** (Màu xanh).

* * * * *

### GIAI ĐOẠN 3: CẤU HÌNH CLOUDFRONT (CDN & FAILOVER)

#### Bước 3.1: Tạo Distribution

1.  Vào **CloudFront** > **Create distribution**.

2.  **Origin:**

    -   **Origin domain:** Chọn bucket Singapore (`sgutodolist-frontend-sg.s3...`).

    -   **Origin access:** Chọn **Origin access control settings (recommended)**.

    -   Click **Create control setting**: Name `S3-OAC-HA`, Type `S3`, Sign requests -> Create.

    -   **Name:** `Primary-SG`.

3.  **Default cache behavior:**

    -   **Viewer protocol policy:** Redirect HTTP to HTTPS.

    -   **Allowed HTTP methods:** GET, HEAD.

4.  **Web Application Firewall (WAF):** Chọn **Do not enable** (để tiết kiệm).

5.  **Settings:**

    -   **Alternate domain names (CNAMEs):** Add item -> Nhập `sgutodolist.com` và `www.sgutodolist.com`.

    -   **Custom SSL certificate:** Chọn chứng chỉ ACM bạn vừa tạo ở Giai đoạn 2.

    -   **Default root object:** `index.html`.

6.  Click **Create distribution**.

#### Bước 3.2: Thêm Origin Phụ (Virginia)

1.  Vào Distribution vừa tạo > Tab **Origins**.

2.  Click **Create origin**.

    -   **Origin domain:** Chọn bucket Virginia (`sgutodolist-frontend-us.s3...`).

    -   **Origin access:** Chọn lại cái `S3-OAC-HA` đã tạo lúc nãy.

    -   **Name:** `Failover-US`.

3.  Click **Create origin**.

#### Bước 3.3: Tạo Origin Group (Kích hoạt High Availability)

1.  Vẫn ở tab Origins > Click **Create origin group**.

2.  **Name:** `HA-Group`.

3.  **Origins:**

    -   Add `Primary-SG` (Lên trên - Ưu tiên 1).

    -   Add `Failover-US` (Xuống dưới - Ưu tiên 2).

4.  **Failover criteria:** Tích chọn: **500, 502, 503, 504**.

5.  Click **Create origin group**.

#### Bước 3.4: Cập nhật Behavior

1.  Tab **Behaviors** > Chọn `Default (*)` > **Edit**.

2.  **Origin and origin groups:** Đổi từ `Primary-SG` sang **`HA-Group`**.

3.  Click **Save changes**.

#### Bước 3.5: Cấu hình SPA Routing (Xử lý lỗi 404 React)

1.  Tab **Error pages** > **Create custom error response**.

2.  **Rule 1 (Cho OAC):**

    -   HTTP error code: **403**.

    -   Customize error response: **Yes**.

    -   Response page path: `/index.html`.

    -   HTTP response code: **200**.

3.  **Rule 2 (Cho React Router):**

    -   Tạo thêm 1 cái tương tự cho mã **404**. (Path vẫn là `/index.html`, code 200).

* * * * *

### GIAI ĐOẠN 4: CẤP QUYỀN S3 (POLICY)

CloudFront cần "giấy phép" để lấy file từ 2 bucket kín của bạn.

1.  Trong CloudFront > Tab **Origins**.

2.  Chọn Origin Singapore > Edit > **Copy policy**.

3.  Mở tab mới > S3 Console > Bucket `sgutodolist-frontend-sg`.

4.  Tab **Permissions** > **Bucket Policy** > Edit > Paste > Save.

5.  **Lặp lại với Bucket Virginia:**

    -   Quay lại CloudFront > Chọn Origin Virginia > Edit > Copy policy.

    -   Sang S3 `sgutodolist-frontend-us` > Permissions > Bucket Policy > Paste > Save.

* * * * *

### GIAI ĐOẠN 5: TRỎ DOMAIN CHÍNH THỨC (DNS RECORD)

1.  Vào **Route 53** > **Hosted zones** > `sgutodolist.com`.

2.  **Tạo Record cho Root domain:**

    -   Click **Create record**.

    -   Record name: (để trống).

    -   Type: **A**.

    -   Alias: **Yes** (Gạt nút sang phải).

    -   Route traffic to: **Alias to CloudFront distribution**.

    -   Choose distribution: Chọn cái CloudFront domain (ví dụ `d123...cloudfront.net`).

    -   Click **Create records**.

3.  **Tạo Record cho WWW:**

    -   Làm tương tự, nhưng Record name điền `www`.

* * * * *

### GIAI ĐOẠN 6: DEPLOY & KIỂM TRA

#### Bước 6.1: Build & Deploy

Tại máy tính của bạn (trong folder project React):

Bash

```
# 1. Build ra folder production
npm run build

# 2. Upload lên bucket CHÍNH (Singapore)
# Lưu ý: Chỉ cần upload lên Sing, AWS sẽ tự copy sang US
aws s3 sync build/ s3://sgutodolist-frontend-sg --delete

# 3. Xóa cache CloudFront để user thấy code mới ngay
aws cloudfront create-invalidation --distribution-id <ID_CUA_BAN> --paths "/*"

```

#### Bước 6.2: Kiểm tra

1.  Truy cập `https://sgutodolist.com`.

2.  Thử reload trang (F5) ở các đường dẫn con (ví dụ `/tasks`) xem có bị lỗi 404 không.

3.  Kiểm tra Replication: Vào S3 Console bucket Virginia xem file đã tự động xuất hiện chưa (thường mất 15s - 1 phút).


