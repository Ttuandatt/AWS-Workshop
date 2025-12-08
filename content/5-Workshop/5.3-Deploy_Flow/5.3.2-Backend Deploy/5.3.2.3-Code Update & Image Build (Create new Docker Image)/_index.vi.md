+++
title = "Cập Nhật Code & Xây Dựng Image (Tạo Docker Image Mới)"
weight = 3
chapter = false
pre = " <b> 5.3.2.3. </b> "
alwaysopen = true
+++

Giai đoạn này tập trung vào việc điều chỉnh mã nguồn ứng dụng cho môi trường cloud. Chúng ta sẽ cấu hình Cross-Origin Resource Sharing (CORS) cho reactive API Gateway, tái cấu trúc các cấu hình ứng dụng để sử dụng biến môi trường động, và cuối cùng xây dựng và đẩy tất cả các Docker image của microservices lên Amazon Elastic Container Registry (ECR).

**Mục tiêu:**

1. Cấu hình code để chạy mượt mà cả trên môi trường Local (Docker Compose) và Cloud (AWS ECS).  
2. Tạo Repositories trên AWS ECR.  
3. Xây dựng và đẩy tất cả 6 Docker image lên AWS.

* * * * *

### 1. TẠO REPOSITORIES TRÊN AWS ECR

Trước khi đẩy image, bạn cần tạo một "repository" cho từng service. Chạy các lệnh sau trong Terminal (PowerShell hoặc Git Bash):

```bash
# Auth Service
aws ecr create-repository --repository-name auth-service --region ap-southeast-1

# User Service
aws ecr create-repository --repository-name user-service --region ap-southeast-1

# Taskflow Service
aws ecr create-repository --repository-name taskflow-service --region ap-southeast-1

# Notification Service
aws ecr create-repository --repository-name notification-service --region ap-southeast-1

# API Gateway
aws ecr create-repository --repository-name api-gateway --region ap-southeast-1

# AI Model Service
aws ecr create-repository --repository-name ai-model-service --region ap-southeast-1
```


### 2. XÂY DỰNG VÀ ĐẨY IMAGE
Thực hiện các lệnh này từ thư mục gốc của dự án (todolist-backend).

1. Đăng nhập Docker vào AWS: (Thay 031133710884 bằng AWS Account ID của bạn nếu khác)

```bash
aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com
```

2. Xây dựng & đẩy từng service trong 6 service:

Service 1: API Gateway

```bash
cd api-gateway
docker build -t api-gateway:latest .
docker tag api-gateway:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/api-gateway:latest
docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/api-gateway:latest
cd ..
```

Service 2: Auth Service

```bash
cd auth-service
docker build -t auth-service:latest .
docker tag auth-service:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/auth-service:latest
docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/auth-service:latest
cd ..
```

Service 3: User Service

```bash
cd user-service
docker build -t user-service:latest .
docker tag user-service:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/user-service:latest
docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/user-service:latest
cd ..
```

Service 4: Taskflow Service

```bash
cd taskflow-service
docker build -t taskflow-service:latest .
docker tag taskflow-service:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/taskflow-service:latest
docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/taskflow-service:latest
cd ..
```

Service 5: Notification Service

```bash
cd notification-service
docker build -t notification-service:latest .
docker tag notification-service:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/notification-service:latest
docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/notification-service:latest
cd ..
```

Service 6: AI Model Service (Thư mục tên là model)

```bash
cd model
docker build -t ai-model-service:latest .
docker tag ai-model-service:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/ai-model-service:latest
docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/ai-model-service:latest
cd ..
```

KẾT QUẢ MONG ĐỢI
Sau khi chạy tất cả lệnh trên, bạn có thể xác minh bằng cách chạy:

```bash
aws ecr describe-repositories --region ap-southeast-1
```

Nếu thấy danh sách 6 repository và không có lỗi trong quá trình push, bạn đã hoàn tất giai đoạn này.

---

<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;"> 
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.2-Infrastructure & ALB Setup (RDS, Redis, Cloud Map, ALB Routing)" %}}" style="text-decoration: none; font-weight: bold;"> 
⬅ BƯỚC 2: Thiết Lập Hạ Tầng & ALB 
</a> 
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.4-Task Definitions Creation (Configure settings, FIX environment variables)" %}}" style="text-decoration: none; font-weight: bold;"> 
BƯỚC 4: Tạo Task Definitions ➡ 
</a> 
</div>