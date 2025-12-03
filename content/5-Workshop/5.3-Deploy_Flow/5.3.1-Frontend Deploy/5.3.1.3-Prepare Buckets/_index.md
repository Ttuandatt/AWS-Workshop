+++
title = "Prepare Buckets"
weight = 3
chapter = false
pre = " <b> 5.3.1.3.  </b> "
+++

3\. PREPARE BUCKETS (SINGAPORE + VIRGINIA)
------------------------------------------

### BƯỚC 3.1: Build ReactJS Project

#### 3.1.1. Cấu hình package.json

Mở package.json, thêm/sửa dòng:

```{
    "name": "aws-todolist",    
    "version": "0.1.0",    
    "homepage": ".",    
    "private": true,    
    ...  }
```

#### 3.1.2. Build Project
```bash
npm run build
```

Kiểm tra thư mục build/ có đầy đủ files.

### BƯỚC 3.2: Tạo S3 Bucket Singapore (Primary)

1.  **Chuyển region**: Góc phải trên → **Asia Pacific (Singapore)** ap-southeast-1
    
2.  Vào **S3** > **Create bucket**
    
3.  Điền:
    
    *   **Bucket name**: sgutodolist-frontend-sg
        
    *   **AWS Region**: ap-southeast-1
        
    *   **Block Public Access**: **BỎ CHỌN TẤT CẢ 4 ô**
        
    *   Tick xác nhận
        
    *   **Bucket Versioning**: **Enable** ⚠️ BẮT BUỘC
        
    *   **Default encryption**: SSE-S3
        
4.  **Create bucket**

{{< figurecaption src="/images/fe4_1.jpg" caption="Hình 4.1. Tạo S3 Bucket Singapore (Primary)">}}

{{< figurecaption src="/images/fe4_2.jpg" caption="Hình 4.2. Tạo S3 Bucket Singapore (Primary)">}}

{{< figurecaption src="/images/fe4_3.jpg" caption="Hình 4.3. Tạo S3 Bucket Singapore (Primary)">}}

    

### BƯỚC 3.3: Upload Files vào Bucket Singapore

1.  Click vào bucket sgutodolist-frontend-sg
    
2.  Click **Upload**
    
3.  Click **Add files** và **Add folder**
    
4.  Chọn **TẤT CẢ** nội dung trong build/:
    
    *   index.html
        
    *   Folder static/
        
    *   favicon.ico, manifest.json, robots.txt, etc.
        
5.  Click **Upload**
    

### BƯỚC 3.4: Static Website Hosting (Singapore)

1.  Tab **Properties**
    
2.  **Static website hosting** > **Edit**
    
3.  **Enable**
    
4.  **Index document**: index.html
    
5.  **Error document**: index.html
    
6.  **Save changes**

{{< figurecaption src="/images/fe5_1.jpg" caption="Hình 5.1. Static Website Hosting (Singapore)">}}

{{< figurecaption src="/images/fe5_2.jpg" caption="Hình 5.2. Static Website Hosting (Singapore)">}}

    

### BƯỚC 3.5: Bucket Policy (Singapore)

1.  Tab **Permissions** > **Bucket policy** > **Edit**
    
2.  Paste:
```bash
    {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::sgutodolist-frontend-sg/*"
        }
    ]
}
```

3. **Save changes**

{{< figurecaption src="/images/fe6.jpg" caption="Hình 6. Bucket Policy (Singapore)">}}

### BƯỚC 3.6: Tạo S3 Bucket Virginia (Replica)

1.  **Chuyển region**: **US East (N. Virginia)** us-east-1
    
2.  **S3** > **Create bucket**
    
3.  Điền:
    
    *   **Bucket name**: sgutodolist-frontend-us
        
    *   **AWS Region**: us-east-1
        
    *   **Block Public Access**: **BỎ CHỌN TẤT CẢ**
        
    *   Tick xác nhận
        
    *   **Bucket Versioning**: **Enable** ⚠️ BẮT BUỘC
        
4.  **Create bucket**
    
{{< figurecaption src="/images/fe7_1.jpg" caption="Hình 7.1. Tạo S3 Bucket Virginia (Replica)">}}

{{< figurecaption src="/images/fe7_2.jpg" caption="Hình 7.2. Tạo S3 Bucket Virginia (Replica)">}}

{{< figurecaption src="/images/fe7_3.jpg" caption="Hình 7.3. Tạo S3 Bucket Virginia (Replica)">}}


### BƯỚC 3.7: Static Website Hosting (Virginia)

1.  Tab **Properties** > **Static website hosting** > **Edit**
    
2.  **Enable**
    
3.  Index/Error: index.html
    
4.  **Save changes**
    
{{< figurecaption src="/images/fe8_1.jpg" caption="Hình 8.1. Static Website Hosting (Virginia)">}}

{{< figurecaption src="/images/fe8_2.jpg" caption="Hình 8.2. Static Website Hosting (Virginia)">}}


### BƯỚC 3.8: Bucket Policy (Virginia)

1.  Tab **Permissions** > **Bucket policy** > **Edit**
    
2.  Paste:
```bash
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::sgutodolist-frontend-us/*"
        }
    ]
}
```
3.  **Save changes**

{{< figurecaption src="/images/fe9.jpg" caption="Hình 9. Bucket Policy (Virginia)">}}

    

✅ **PHASE 1 COMPLETE** - 2 buckets ready với versioning ON

---

<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.2-Setup Domain & Route 53" %}}" style="text-decoration: none; font-weight: bold;">
⬅ BƯỚC 2: Setup Domain & Route 53
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.4-Cross-Region Replication" %}}" style="text-decoration: none; font-weight: bold;">
BƯỚC 4: Cross-Region Replication ➡
</a>
</div>