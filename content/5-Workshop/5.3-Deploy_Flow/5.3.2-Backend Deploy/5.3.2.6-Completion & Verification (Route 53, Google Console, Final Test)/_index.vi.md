+++
title = "Hoàn thiện & Xác nhận (Route 53, Google Console, Kiểm thử cuối)"
weight = 6
chapter = false
pre = " <b> 5.3.2.6. </b> "
alwaysopen = true
+++

Giai đoạn cuối cùng này sẽ kết nối backend đã triển khai với tên miền công cộng (public domain) và xác minh toàn bộ chức năng từ đầu đến cuối (end-to-end).

### Cấu hình DNS

**Bước 1: Tạo bản ghi A (A Record) trên Route 53**

1.  Điều hướng đến **Route 53** → **Hosted zones** (Vùng lưu trữ).

2.  Chọn hosted zone `sgutodolist.com`.

3.  Nhấn vào **Create record** (Tạo bản ghi).

4.  Cấu hình bản ghi:

    -   **Record name**: `api`

    -   **Record type**: A

    -   **Alias**: Bật (Enable)

    -   **Route traffic to**: Alias to Application Load Balancer

    -   **Region**: Asia Pacific (Singapore)

    -   **Load balancer**: Chọn `sgu-alb`

5.  Nhấn **Create records**.

**Lan truyền DNS:** Chờ khoảng 2-5 phút để các thay đổi DNS được cập nhật trên toàn hệ thống.

**Xác minh:**

bash

```
nslookup sgutodolist.com
# Kết quả trả về danh sách địa chỉ IP của ALB

```

* * * * *

### Cập nhật Cấu hình Google OAuth

**Bước 1: Cập nhật các URI chuyển hướng được ủy quyền (Authorized Redirect URIs)**

1.  Truy cập **Google Cloud Console** (console.cloud.google.com).

2.  Điều hướng đến **APIs & Services** → **Credentials** (Thông tin xác thực).

3.  Chọn **OAuth 2.0 Client ID** của dự án.

4.  Trong phần **Authorized redirect URIs**, thêm đường dẫn sau:

```
   https://sgutodolist.com/api/auth/login/oauth2/code/google

```

1.  Nhấn **Save** (Lưu).

**Lưu ý:** Giữ nguyên các URI localhost hiện có để phục vụ cho việc phát triển cục bộ (local).

* * * * *

### Xác minh Tình trạng Hệ thống (System Health)

Thực hiện các bài kiểm tra sau để xác nhận việc triển khai thành công:

#### Test 1: Kiểm tra Health Check của API Gateway

bash

```
curl https://sgutodolist.com/actuator/health

```

**Phản hồi mong đợi:**

json

```
{
  "status": "UP"
}

```

#### Test 2: Kiểm tra Health Check của từng Service

bash

```
# Auth Service
curl https://sgutodolist.com/api/auth/actuator/health

# User Service
curl https://sgutodolist.com/api/user/actuator/health

# Taskflow Service
curl https://sgutodolist.com/api/taskflow/actuator/health

# Notification Service
curl https://sgutodolist.com/api/notification/actuator/health

```

Tất cả đều phải trả về `{"status":"UP"}`.

#### Test 3: Xác minh Service Discovery

Từ Bastion Host, kiểm tra khả năng phân giải DNS nội bộ:

bash

```
# SSH vào Bastion
ssh -i sgutodolist-key.pem ec2-user@[BASTION-IP]

# Test phân giải DNS
nslookup auth.sgu.local
nslookup user.sgu.local
nslookup taskflow.sgu.local
nslookup notification.sgu.local
nslookup ai-model.sgu.local
nslookup kafka.sgu.local

```

Tất cả phải phân giải ra địa chỉ IP của các ECS task nội bộ.

#### Test 4: Kết nối Cơ sở dữ liệu

Xác minh các service có thể kết nối tới RDS:

1.  Kiểm tra **CloudWatch Logs** cho bất kỳ service nào.

2.  Tìm các thông báo kết nối cơ sở dữ liệu thành công.

3.  Đảm bảo không có lỗi kết nối (connection errors) trong log khởi động.

#### Test 5: Kết nối Redis

bash

```
# Từ Bastion Host
redis-cli -h [REDIS-ENDPOINT] ping
# Phản hồi mong đợi: PONG

```

#### Test 6: Luồng xác thực End-to-End

1.  Truy cập frontend tại `https://sgutodolist.com`.

2.  Nhấn nút "Sign in with Google".

3.  Hoàn tất quy trình đăng nhập OAuth.

4.  Xác minh đăng nhập thành công và token được cấp phát.

5.  Xác minh thông tin hồ sơ người dùng (profile) được tải chính xác.

* * * * *

### Thiết lập Mức hiệu năng cơ sở (Performance Baseline)

Ghi lại các chỉ số hiệu năng ban đầu để làm chuẩn so sánh sau này:

**Đo lường thời gian phản hồi (Response Time):**

bash

```
# Thời gian phản hồi API Gateway
time curl -o /dev/null -s https://sgutodolist.com/actuator/health

# Thời gian phản hồi Auth service
time curl -o /dev/null -s https://sgutodolist.com/api/auth/actuator/health

```

**Các chỉ số CloudWatch cần giám sát:**

-   ECS Task CPU Utilization (Mức sử dụng CPU của Task)

-   ECS Task Memory Utilization (Mức sử dụng RAM của Task)

-   ALB Target Response Time (Thời gian phản hồi mục tiêu ALB)

-   ALB Request Count (Số lượng yêu cầu tới ALB)

-   RDS CPU Utilization

-   Redis CPU Utilization

* * * * *

### Danh sách kiểm tra bảo mật sau triển khai

-   [ ] Tất cả các biến môi trường nhạy cảm đã được bảo mật (không lưu trong version control).

-   [ ] Mật khẩu cơ sở dữ liệu đáp ứng yêu cầu về độ phức tạp.

-   [ ] Security Group tuân thủ nguyên tắc đặc quyền tối thiểu (least privilege).

-   [ ] Chứng chỉ SSL/TLS hợp lệ và đã bật tự động gia hạn.

-   [ ] Bastion Host chỉ có thể truy cập từ các địa chỉ IP được ủy quyền.

-   [ ] Cấu hình thời gian lưu trữ (retention) cho CloudWatch Logs.

-   [ ] Các cảnh báo ngân sách (AWS Budget alerts) đang hoạt động.

* * * * *

### Danh sách kiểm tra triển khai cuối cùng

**Cơ sở hạ tầng:**

-   [ ] VPC và các subnet hoạt động bình thường.

-   [ ] Cả 4 Security Group được cấu hình chính xác.

-   [ ] RDS database có thể truy cập và đã khởi tạo dữ liệu.

-   [ ] Redis cache hoạt động tốt.

-   [ ] Dịch vụ Kafka đang chạy.

-   [ ] ALB đang hoạt động với các target ở trạng thái healthy.

**Ứng dụng:**

-   [ ] Cả 6 service đã được deploy và đang chạy.

-   [ ] Service Discovery hoạt động tốt.

-   [ ] Các quy tắc định tuyến (routing rules) của ALB hoạt động chính xác.

-   [ ] Health check đều vượt qua (passing).

-   [ ] CloudWatch Logs đang thu thập dữ liệu.

**Tích hợp:**

-   [ ] Bản ghi DNS trỏ về ALB chính xác.

-   [ ] Chứng chỉ SSL hợp lệ.

-   [ ] Google OAuth đã được cấu hình.

-   [ ] Frontend có thể giao tiếp với Backend.

-   [ ] Luồng xác thực (Authentication flow) hoạt động.

**Giám sát:**

-   [ ] CloudWatch Dashboard đã được tạo.

-   [ ] Cảnh báo ngân sách đã cấu hình.

-   [ ] Đã ghi lại hiệu năng cơ sở (baseline).

* * * * *

### Các hạn chế đã biết và Cải tiến trong tương lai

**Các ràng buộc kiến trúc hiện tại:**

1.  **Single-AZ Database**: RDS được triển khai trên một Availability Zone duy nhất để tối ưu chi phí.

2.  **Single-Node Redis**: Không có cơ chế tự động chuyển đổi dự phòng (failover) cho lớp cache.

3.  **Single-Node Kafka**: Chưa đạt chuẩn production cho các kịch bản lưu lượng cao.

4.  **Public Subnet ECS Tasks**: Đánh đổi về bảo mật để tiết kiệm chi phí (không dùng NAT Gateway).

**Các cải tiến đề xuất cho Production:**

1.  Bật triển khai RDS Multi-AZ.

2.  Triển khai Redis Cluster Mode với nhiều bản sao (replicas).

3.  Triển khai Kafka với nhiều broker trên các AZ khác nhau.

4.  Thêm NAT Gateway và di chuyển các ECS task vào private subnet.

5.  Triển khai AWS WAF trên ALB để chống tấn công DDoS.

6.  Bật tính năng Auto Scaling cho ECS Service.

7.  Xây dựng quy trình CI/CD để triển khai tự động.

* * * * *

Hướng dẫn Khắc phục sự cố (Troubleshooting)
-------------------------------------------

### Các lệnh chẩn đoán nhanh

bash

```
# Kiểm tra trạng thái ECS service
aws ecs describe-services --cluster [CLUSTER-NAME] --services [SERVICE-NAME] --region ap-southeast-1

# Kiểm tra trạng thái task
aws ecs describe-tasks --cluster [CLUSTER-NAME] --tasks [TASK-ARN] --region ap-southeast-1

# Xem log gần nhất
aws logs tail /ecs/[SERVICE-NAME] --follow --region ap-southeast-1

# Kiểm tra sức khỏe target group
aws elbv2 describe-target-health --target-group-arn [TG-ARN] --region ap-southeast-1

```

### Quy trình Rollback khẩn cấp

Nếu việc triển khai thất bại:

1.  **Xác định service bị lỗi**:

bash

```
   aws ecs list-services --cluster [CLUSTER-NAME] --region ap-southeast-1

```

1.  **Cập nhật service về phiên bản task definition trước đó**:

bash

```
   aws ecs update-service\
     --cluster [CLUSTER-NAME]\
     --service [SERVICE-NAME]\
     --task-definition [TASK-DEF-FAMILY]:[PREVIOUS-REVISION]\
     --region ap-southeast-1

```

1.  **Buộc triển khai lại (Force new deployment)**:

bash

```
   aws ecs update-service\
     --cluster [CLUSTER-NAME]\
     --service [SERVICE-NAME]\
     --force-new-deployment\
     --region ap-southeast-1

```

* * * * *

Tiêu chí thành công
-------------------

Việc triển khai được coi là thành công khi:

1.  ✅ Tất cả 6 ECS services hiển thị trạng thái: RUNNING

2.  ✅ Tất cả 5 Target Group hiển thị trạng thái: Healthy

3.  ✅ `https://sgutodolist.com/actuator/health` trả về HTTP 200

4.  ✅ Frontend tại `https://sgutodolist.com` có thể xác thực qua Google OAuth

5.  ✅ CloudWatch Logs không hiển thị lỗi nghiêm trọng nào

6.  ✅ Tất cả services có thể truy cập được qua DNS nội bộ (*.sgu.local)
* * * * *

<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;"> <a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.5-Services Deployment" %}}" style="text-decoration: none; font-weight: bold;"> ⬅ BƯỚC 5: Code Update & Image Build </a> <a href="" style="text-decoration: none; font-weight: bold;"></a> </div>