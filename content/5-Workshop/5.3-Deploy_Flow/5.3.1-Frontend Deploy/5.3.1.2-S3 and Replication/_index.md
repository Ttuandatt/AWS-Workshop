+++
title = "S3 and Replication"
weight = 2
chapter = false
pre = " <b> 5.3.1.2.  </b> "
+++



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

{{< figurecaption src="/images/fe1.1_1.jpg" caption="">}}

{{< figurecaption src="/images/fe1.1_2.jpg" caption="">}}

{{< figurecaption src="/images/fe1.1_3.jpg" caption="">}}



#### Bước 1.2: Tạo Bucket Phụ (Virginia)

1.  Đổi Region sang **US East (N. Virginia)**.

2.  Click **Create bucket**.

    -   **Bucket name:** `sgutodolist-frontend-us`.

    -   **Block Public Access:** ✅ **Block all public access**.

    -   **Bucket Versioning:** ✅ **Enable**.

3.  Click **Create bucket**.

Sau bước **1.1 & 1.2**, ta có 2 S3 buckets

{{< figurecaption src="/images/fe1.1_1.2.jpg" caption="">}}

#### Bước 1.3: Cấu hình Replication (Tự động Sync)

1.  Quay lại bucket **Singapore** (`sgutodolist-frontend-sg`).

2.  Tab **Management** > Mục **Replication rules** > Click **Create replication rule**.

    -   **Rule name:** `SyncToUS`.

    -   **Status:** Enabled.

    -   **Source bucket:** Apply to all objects in the bucket.

    -   **Destination:** Choose a bucket in this account > Chọn `sgutodolist-frontend-us` (nhớ chọn region us-east-1 để tìm thấy).

    -   **IAM Role:** Chọn **Create new role** (AWS tự tạo quyền cho bạn).

3.  Click **Save**. Khi hỏi "Replicate existing objects?", chọn **No** (vì bucket đang rỗng).

{{< figurecaption src="/images/fe1.3_1.jpg" caption="">}}

{{< figurecaption src="/images/fe1.3_2.jpg" caption="">}}

{{< figurecaption src="/images/fe1.3_3.jpg" caption="">}}

{{< figurecaption src="/images/fe1.3_4.jpg" caption="">}}

* * * * *

<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.1-Prerequisites" %}}" style="text-decoration: none; font-weight: bold;">
⬅ BƯỚC 1: Prerequisites
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.3-Route 53 and ACM" %}}" style="text-decoration: none; font-weight: bold;">
BƯỚC 3: Route 53 and ACM ➡
</a>
</div>