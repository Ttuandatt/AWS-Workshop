+++
title = "DNS Record"
weight =  6
chapter = false
pre = " <b> 5.3.1.6. </b>"
+++

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

![](/AWS-Workshop/images/fe5_1.jpg)


3.  **Tạo Record cho WWW:**

    -   Làm tương tự, nhưng Record name điền `www`.

![](/AWS-Workshop/images/fe5_2.jpg)

* * * * *
<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.5-S3 Policy" %}}" style="text-decoration: none; font-weight: bold;">
⬅ BƯỚC 5: S3 Policy
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.7-Deploy and Test" %}}" style="text-decoration: none; font-weight: bold;">
BƯỚC 7: Deploy & Test ➡
</a>
</div>
