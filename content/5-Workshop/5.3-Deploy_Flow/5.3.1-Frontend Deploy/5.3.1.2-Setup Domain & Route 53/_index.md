+++
title = "Setup Domain & Route 53"
weight = 2
chapter = false
pre = " <b> 5.3.1.2.  </b> "
+++

2\. SETUP DOMAIN & ROUTE 53
---------------------------

### BƯỚC 2.1: Tạo Hosted Zone trong Route 53

1.  Đăng nhập **AWS Console**
    
2.  Tìm và vào **Route 53**
    
3.  Click **Hosted zones** (menu bên trái)
    
4.  Click **Create hosted zone**
    
5.  Điền thông tin:
    
    *   **Domain name**: sgutodolist.com
        
    *   **Type**: **Public hosted zone**
        
6.  Click **Create hosted zone**

{{< figurecaption src="/images/fe1.jpg" caption="Hình 1. Tạo Hosted Zone trong Route 53">}}
    

### BƯỚC 2.2: Lấy Name Servers

Sau khi tạo xong, trong tab **Records**, ta sẽ thấy **4 Name Servers (NS)**:
```
ns-1538.awsdns-00.co.uk
ns-1374.awsdns-43.org
ns-172.awsdns-21.com
ns-547.awsdns-04.net
```

{{< figurecaption src="/images/fe2.jpg" caption="Hình 2. Các Name Servers">}}


Copy 4 Name Servers này

### BƯỚC 2.3: Cập nhật Name Servers ở Nhà đăng ký

1.  Đăng nhập vào trang quản lý tên miền (GoDaddy, Namecheap, PA Vietnam...)
    
2.  Tìm phần **Name Servers** hoặc **DNS Settings**
    
3.  Chuyển từ **Default** sang **Custom Name Servers**
    
4.  Nhập **4 Name Servers** từ Route 53
    
5.  **Save changes**

{{< figurecaption src="/images/fe3.jpg" caption="Hình 3. Cập nhật các Name Servers ở trang quản lý tên miền">}}

    

**Lưu ý:** DNS propagation mất 24-48 giờ

---

<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.1-Prerequisites" %}}" style="text-decoration: none; font-weight: bold;">
⬅ BƯỚC 1: Prerequisites
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.3-Prepare Buckets" %}}" style="text-decoration: none; font-weight: bold;">
BƯỚC 3: Prepare Buckets ➡
</a>
</div>