+++
title = "DNS Record"
weight = 6
chapter = false
pre = " <b> 5.3.1.6.  </b> "
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

{{< figurecaption src="/images/fe5_1.jpg" caption="">}}


3.  **Tạo Record cho WWW:**

    -   Làm tương tự, nhưng Record name điền `www`.

{{< figurecaption src="/images/fe5_2.jpg" caption="">}}

* * * * *