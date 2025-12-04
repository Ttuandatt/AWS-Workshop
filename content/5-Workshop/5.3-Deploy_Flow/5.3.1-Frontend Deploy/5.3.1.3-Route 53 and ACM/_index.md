+++
title = "Route 53 and ACM"
weight = 3
chapter = false
pre = " <b> 5.3.1.3.  </b> "
+++


### GIAI ĐOẠN 2: CHUẨN BỊ TÊN MIỀN & BẢO MẬT (ROUTE 53 & ACM)

Trước khi tạo CloudFront, ta cần chuẩn bị Chứng chỉ bảo mật (SSL) và DNS.

#### Bước 2.1: Tạo Hosted Zone (Nếu chưa có)

1.  Vào **Route 53** > **Hosted zones**.

2.  Nếu chưa có domain `sgutodolist.com` trong list:

    -   Click **Create hosted zone**.

    -   Domain name: `sgutodolist.com`.

    -   Type: Public hosted zone.

    -   Click **Create**.

{{< figurecaption src="/images/fe2.1_1.jpg" caption="">}}

3. Cập nhật Nameservers bên nhà cung cấp tên miền vì project sử dụng tên miền mua ở nhà cung cấp khác
    - Khi nhấp vào Hosted zone vừa tạo, ta sẽ thấy được 4 name servers có dạng
    ```
    ns-1538.awsdns-00.co.uk.
    ns-1374.awsdns-43.org.
    ns-172.awsdns-21.com.
    ns-547.awsdns-04.net.
    ```
    - Dùng 4 name servers đó để cập nhật từ trang quản lý domain mà ta thuê của nhà cung cấp khác

{{< figurecaption src="/images/fe2.1_2.jpg" caption="">}}



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

{{< figurecaption src="/images/fe2.2_1.jpg" caption="">}}

{{< figurecaption src="/images/fe2.2_2.jpg" caption="">}}

{{< figurecaption src="/images/fe2.2_3.jpg" caption="">}}


* * * * *