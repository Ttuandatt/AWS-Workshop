+++
title = "Cross-Region Replication"
weight = 4
chapter = false
pre = " <b> 5.3.1.4.  </b> "
+++

4\. SETUP CROSS-REGION REPLICATION (CRR)
----------------------------------------

### BƯỚC 4.1: Tạo IAM Policy

1.  Vào **IAM** > **Policies** > **Create policy**
    
2.  Tab **JSON**, paste:
```bash
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "S3ReplicationSourcePermissions",
            "Effect": "Allow",
            "Action": [
                "s3:GetReplicationConfiguration",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::sgutodolist-frontend-sg"
            ]
        },
        {
            "Sid": "S3ReplicationSourceObjectPermissions",
            "Effect": "Allow",
            "Action": [
                "s3:GetObjectVersionForReplication",
                "s3:GetObjectVersionAcl",
                "s3:GetObjectVersionTagging",
                "s3:GetObjectRetention",
                "s3:GetObjectLegalHold"
            ],
            "Resource": [
                "arn:aws:s3:::sgutodolist-frontend-sg/*"
            ]
        },
        {
            "Sid": "S3ReplicationDestinationPermissions",
            "Effect": "Allow",
            "Action": [
                "s3:ReplicateObject",
                "s3:ReplicateDelete",
                "s3:ReplicateTags",
                "s3:ObjectOwnerOverrideToBucketOwner"
            ],
            "Resource": [
                "arn:aws:s3:::sgutodolist-frontend-us/*"
            ]
        }
    ]
}
```

3.  **Next**
    
4.  **Policy name**: S3-CRR-Policy-Frontend
    
5.  **Create policy**
    

{{< figurecaption src="/images/fe10_1.jpg" caption="Hình 10.1. Tạo IAM Policy cho S3 Replication">}}

{{< figurecaption src="/images/fe10_2.jpg" caption="Hình 10.2. Tạo IAM Policy cho S3 Replication">}}

Kết quả policy được tạo

{{< figurecaption src="/images/fe10_3.jpg" caption="Hình 10.3. Tạo IAM Policy cho S3 Replication">}}




### BƯỚC 4.2: Tạo IAM Role

1.  **IAM** > **Roles** > **Create role**
    
2.  **Trusted entity type**: **Custom trust policy**
    
3.  Paste:
```bash
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "s3.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

4.  **Next**
    
5.  Search và chọn: S3-CRR-Policy-Frontend
    
6.  **Next**
    
7.  **Role name**: S3-CRR-Role-Frontend
    
8.  **Create role**

{{< figurecaption src="/images/fe11_1.jpg" caption="Hình 11.1. Tạo IAM Role cho S3 Replication">}}

{{< figurecaption src="/images/fe11_2.jpg" caption="Hình 11.2. Tạo IAM Role cho S3 Replication">}}

{{< figurecaption src="/images/fe11_3.jpg" caption="Hình 11.3. Tạo IAM Role cho S3 Replication">}}

{{< figurecaption src="/images/fe11_4.jpg" caption="Hình 11.4. Tạo IAM Role cho S3 Replication">}}


### BƯỚC 4.3: Tạo Replication Rule

1.  **Chuyển về region Singapore** (ap-southeast-1)
    
2.  **S3** > Bucket sgutodolist-frontend-sg > Tab **Management**
    
3.  **Replication rules** > **Create replication rule**
    
4.  Cấu hình:
    
    *   **Name**: frontend-sg-to-us
        
    *   **Status**: **Enabled**
        
    *   **Source bucket**: Apply to all objects
        
    *   **Destination**: Bucket sgutodolist-frontend-us
        
    *   **IAM role**: S3-CRR-Role-Frontend
        
    *   **Storage class**: Use source object class
        
    *   **Additional options**: ✅ Replication metrics
        
5.  **Save**

{{< figurecaption src="/images/fe12_1.jpg" caption="Hình 12.1. Tạo Replication Rule cho S3 Replication">}}

{{< figurecaption src="/images/fe12_2.jpg" caption="Hình 12.2. Tạo Replication Rule cho S3 Replication">}}

{{< figurecaption src="/images/fe12_3.jpg" caption="Hình 12.3. Tạo Replication Rule cho S3 Replication">}}

{{< figurecaption src="/images/fe12_4.jpg" caption="Hình 12.4. Tạo Replication Rule cho S3 Replication">}}

    


### BƯỚC 4.4: Replicate Existing Objects (Sao chép dữ liệu cũ)

**Trường hợp 1: Có popup "Replicate existing objects?"**

1.  Tại popup, chọn **"Yes, replicate existing objects"**.
2.  Click **Submit**.
3.  Hệ thống chuyển sang màn hình **Create Batch Operations job**. Cấu hình như sau:
    * **Job run options:** Chọn **Automatically run the job when it's ready** (Để job tự chạy ngay khi chuẩn bị xong).
    * **Completion report:**
        * Tích chọn **Generate completion report**.
        * **Scope:** Chọn **Failed tasks only** (Chỉ báo cáo file lỗi).
        * **Destination:** Chọn **Browse S3** > Chọn bucket gốc `sgutodolist-frontend-sg`.
    * **Permissions (Quan trọng):**
        * Tại mục **IAM role**, chọn **Create new role**.
        * > **Lưu ý:** Không chọn lại role `S3-CRR-Role-Frontend` cũ vì Batch Job cần một role riêng có quyền Batch Operations.
4.  Cuộn xuống cuối và click **Save**.

{{< figurecaption src="/images/fe13_1.jpg" caption="Hình 13.1. Replicate Existing Objects (Sao chép dữ liệu cũ)">}}

{{< figurecaption src="/images/fe13_2.jpg" caption="Hình 13.2. Replicate Existing Objects (Sao chép dữ liệu cũ)">}}

{{< figurecaption src="/images/fe13_3.jpg" caption="Hình 13.3. Replicate Existing Objects (Sao chép dữ liệu cũ)">}}



**Trường hợp 2: KHÔNG có popup (Làm thủ công)**

1.  Vào **S3** > **Batch Operations** (Region Singapore).
2.  Chọn **Create job**.
3.  **Manifest:**
    * **Region:** Singapore.
    * **Manifest format:** S3 Replication configuration.
    * **Source bucket:** `sgutodolist-frontend-sg`.
4.  **Operation:** Chọn **Replicate**.
5.  **Configure:**
    * **Description:** `Replicate frontend SG to US`.
    * **Permissions:** Chọn **Create a new role**.
6.  Click **Create job**.
7.  Nếu job ở trạng thái *Awaiting confirmation*, chọn job đó và click **Run job**.
    

**Nếu KHÔNG có popup, tạo Batch Job thủ công:**

1.  **S3** > **Batch Operations** (Region Singapore)
    
2.  **Create job**
    
3.  **Manifest**:
    
    *   Region: Singapore
        
    *   Format: **S3 Replication configuration**
        
    *   Bucket: sgutodolist-frontend-sg
        
    *   Replication configuration: frontend-sg-to-us
        
4.  **Next**
    
5.  **Operation**: Replicate
    
6.  **Next**
    
7.  **Description**: Replicate frontend SG to US
    
8.  **Completion report**: Enable
    
    *   Destination: sgutodolist-frontend-sg
        
9.  **Permissions**: Create a new role
    
10.  **Next** > **Create job**
    
11.  Click vào job > **Confirm and run**
    

### BƯỚC 4.5: Verify Replication

**Đợi job complete (5-15 phút), sau đó:**

1.  Chuyển sang **region Virginia**
    
2.  Vào bucket sgutodolist-frontend-us
    
3.  Kiểm tra files đã xuất hiện
    
4.  Compare object count (Singapore vs Virginia phải bằng nhau)

Kết quả Replicate:

{{< figurecaption src="/images/fe14_1.jpg" caption="Hình 14.1. Kết quả Batch Operation Jobs">}}

{{< figurecaption src="/images/fe14_2.jpg" caption="Hình 14.2. S3 ở region Virginia đã Replicate thành công">}}

    

✅ **PHASE 2 COMPLETE** - CRR setup xong!