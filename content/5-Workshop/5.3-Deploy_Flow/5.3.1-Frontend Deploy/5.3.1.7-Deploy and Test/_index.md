+++
title = "Deploy and Test"
weight = 7
chapter = false
pre = " <b> 5.3.1.7.  </b> "
+++

### GIAI ĐOẠN 6: DEPLOY & KIỂM TRA

#### Bước 6.1: Build & Deploy

Tại máy tính local (trong folder project React):


```bash
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

{{< figurecaption src="/images/fe6.2_2.jpg" caption="">}}
