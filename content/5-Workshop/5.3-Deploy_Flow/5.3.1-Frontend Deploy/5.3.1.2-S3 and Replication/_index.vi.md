+++
title = "S3 và Replication"
weight = 2
chapter = false
pre = " <b> 5.3.1.2. </b> "
+++

## GIAI ĐOẠN 1: CHUẨN BỊ STORAGE (S3 & REPLICATION)

Giai đoạn này tập trung vào việc xây dựng lớp lưu trữ bền vững cho frontend và thiết lập cơ chế **đồng bộ dữ liệu tự động giữa các region (Cross-Region Replication)** nhằm phục vụ khả năng **failover và high availability**.

---

### Bước 1.1: Tạo Bucket Chính (Singapore)

1. Truy cập **Amazon S3 Console** và chọn region **Asia Pacific (Singapore)**.

2. Click **Create bucket**.

   - **Bucket name:** `sgutodolist-frontend-sg`
   - **Object Ownership:** ACLs disabled (Recommended)
   - **Block Public Access:** ✅ **Block all public access**  
     (Bucket phải ở chế độ private vì sẽ sử dụng CloudFront OAC.)
   - **Bucket Versioning:** ✅ **Enable**  
     (Bắt buộc để sử dụng Cross-Region Replication.)
   - **Default encryption:** Server-side encryption with Amazon S3 managed keys (SSE-S3)

3. Click **Create bucket**.

![](/AWS-Workshop/images/fe1.1_1.jpg)

![](/AWS-Workshop/images/fe1.1_2.jpg)

![](/AWS-Workshop/images/fe1.1_3.jpg)


---

### Bước 1.2: Tạo Bucket Phụ (N. Virginia)

1. Chuyển region sang **US East (N. Virginia)**.

2. Click **Create bucket**.

   - **Bucket name:** `sgutodolist-frontend-us`
   - **Block Public Access:** ✅ **Block all public access**
   - **Bucket Versioning:** ✅ **Enable**

3. Click **Create bucket**.

Sau khi hoàn thành **Bước 1.1** và **Bước 1.2**, hệ thống sẽ có **2 S3 bucket**:

- Bucket chính: Singapore (`ap-southeast-1`)
- Bucket dự phòng: N. Virginia (`us-east-1`)

![](/AWS-Workshop/images/fe1.1_1.2.jpg)

---

### Bước 1.3: Cấu Hình Replication (Tự Động Đồng Bộ)

1. Quay lại bucket **Singapore** (`sgutodolist-frontend-sg`).

2. Vào tab **Management** → **Replication rules** → Click **Create replication rule**.

   - **Rule name:** `SyncToUS`
   - **Status:** Enabled
   - **Source bucket:** Apply to all objects in the bucket
   - **Destination:** Choose a bucket in this account  
     → Chọn `sgutodolist-frontend-us`  
     (Đảm bảo đang filter region `us-east-1` để bucket hiển thị.)
   - **IAM Role:** Chọn **Create new role**  
     (AWS sẽ tự động tạo IAM Role với đầy đủ quyền cần thiết.)

3. Click **Save**.  
   Khi được hỏi **“Replicate existing objects?”**, chọn **No**  
   (Do bucket hiện tại đang trống.)

![](/AWS-Workshop/images/fe1.3_1.jpg)

![](/AWS-Workshop/images/fe1.3_2.jpg)

![](/AWS-Workshop/images/fe1.3_3.jpg)

![](/AWS-Workshop/images/fe1.3_4.jpg)

---

---

<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.1-Prerequisites" %}}" style="text-decoration: none; font-weight: bold;">
⬅ BƯỚC 1: Điều kiện tiên quyết
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.3-Route 53 and ACM" %}}" style="text-decoration: none; font-weight: bold;">
BƯỚC 3: Route 53 & ACM ➡
</a>
</div>
