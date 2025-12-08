+++
title = "S3 Policy"
weight =  5
chapter = false
pre = " <b> 5.3.1.5.  </b>"
+++


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
<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.4-ClouFront and Failover" %}}" style="text-decoration: none; font-weight: bold;">
⬅ BƯỚC 4: CloudFront và Failover
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.6-DNS Record" %}}" style="text-decoration: none; font-weight: bold;">
BƯỚC 6: DNS Record ➡
</a>
</div>
