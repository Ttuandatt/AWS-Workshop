Chào bạn, đây là **Tài liệu hướng dẫn triển khai CI/CD toàn diện (A-Z)** dành riêng cho dự án SGU TodoList của bạn.

Tài liệu này tổng hợp lại tất cả những gì chúng ta đã tối ưu: **Tự động hóa hoàn toàn, Giao diện đẹp, Báo lỗi thông minh, và Chiến lược nhánh chuẩn.**

* * * * *

### 🏁 QUY TRÌNH TỔNG QUAN

1.  **Cấu hình bảo mật:** Gửi chìa khóa AWS cho GitHub.

2.  **Cấu trúc dự án:** Tạo file workflow `.yml`.

3.  **Nội dung Workflow:** Copy code chuẩn cho 6 services.

4.  **Kích hoạt:** Đẩy code lên GitHub để hệ thống tự chạy.

* * * * *

### BƯỚC 1: CẤU HÌNH BẢO MẬT (SECRETS)

GitHub cần quyền truy cập AWS để thay bạn deploy.

1.  Vào Repository trên GitHub.

2.  Chọn **Settings** > **Secrets and variables** > **Actions**.

3.  Bấm **New repository secret** và thêm 3 biến sau (Lấy từ file `.aws/credentials`):

| **Tên Secret** | **Giá trị (Value)** | **Lưu ý** |
| --- | --- | --- |
| **`AWS_ACCESS_KEY_ID`** | `AKIA...` (hoặc `ASIA...`) | Bắt buộc |
| **`AWS_SECRET_ACCESS_KEY`** | `...` (Chuỗi ký tự dài) | Bắt buộc |
| **`AWS_SESSION_TOKEN`** | `...` (Chuỗi rất dài) | **Bắt buộc** nếu dùng AWS Academy (Learner Lab). Update 4 tiếng/lần. |

* * * * *

### BƯỚC 2: CHUẨN BỊ FILE CẤU HÌNH

Tại thư mục gốc dự án trên máy tính, bạn hãy tạo (hoặc kiểm tra) cấu trúc thư mục như sau:

Plaintext

```
MY-PROJECT/
├── .gitignore               <-- File loại bỏ rác (tạo mới nếu chưa có)
├── .github/
│   └── workflows/           <-- Nơi chứa các file CI/CD
│       ├── deploy-auth.yml
│       ├── deploy-gateway.yml
│       ├── deploy-user.yml
│       ├── deploy-task.yml
│       ├── deploy-noti.yml
│       └── deploy-model.yml

```

**Mẹo nhỏ:** Tạo file `.gitignore` và thêm dòng `*.zip` vào để tránh đẩy file rác lên GitHub.

* * * * *

### BƯỚC 3: NỘI DUNG WORKFLOW (Copy & Paste)

#### 1\. File: `.github/workflows/deploy-apigateway.yml`


```YAML
# ========================================================
# 1. ĐỊNH DANH WORKFLOW
# ========================================================
name: Deploy API Gateway 🌐  # Tên hiển thị của quy trình này trong tab "Actions" trên GitHub.

# ========================================================
# 2. TRIGGER (CÒ KÍCH HOẠT)
# ========================================================
on:
  push:  # Sự kiện kích hoạt: Khi có hành động "Push code" lên...
    branches: [ "main", "pre-production" ]  # ...một trong 2 nhánh này (các nhánh khác push lên sẽ không chạy).
    
    paths:  # BỘ LỌC THÔNG MINH (Quan trọng):
      - 'api-gateway/**'                    # Chỉ chạy khi có file thay đổi trong thư mục 'api-gateway'.
      - '.github/workflows/deploy-gateway.yml'  # Hoặc khi chính file cấu hình này bị sửa đổi.
      # (Giúp tiết kiệm tài nguyên: Sửa auth-service sẽ không kích hoạt deploy api-gateway).

# ========================================================
# 3. BIẾN MÔI TRƯỜNG (GLOBAL VARIABLES)
# ========================================================
env:
  AWS_REGION: ap-southeast-1          # Khu vực máy chủ AWS (Singapore).
  ECR_REPOSITORY: api-gateway         # Tên kho chứa Image trên AWS ECR.
  ECS_SERVICE: api-gateway            # Tên Service đang chạy trên AWS ECS Fargate.
  ECS_CLUSTER: SGUTodolist-Cluster    # Tên Cluster chứa các service.
  WORKING_DIRECTORY: ./api-gateway    # Đường dẫn đến thư mục chứa mã nguồn của service này.

# ========================================================
# 4. JOBS (CÁC CÔNG VIỆC CẦN LÀM)
# ========================================================
jobs:
  deploy:              # ID của công việc (job).
    name: Build & Deploy  # Tên hiển thị của job.
    runs-on: ubuntu-latest  # Môi trường chạy: GitHub sẽ cấp cho bạn 1 máy ảo Ubuntu mới tinh.
    environment: production # Gắn nhãn môi trường (để quản lý Secrets tốt hơn trên GitHub).

    # ====================================================
    # 5. STEPS (CÁC BƯỚC THỰC HIỆN TUẦN TỰ)
    # ====================================================
    steps:
    
    # BƯỚC 1: LẤY CODE VỀ
    - name: Checkout Code
      id: checkout             # Đặt ID để bước xử lý lỗi cuối cùng có thể kiểm tra trạng thái.
      uses: actions/checkout@v3 # Dùng action có sẵn của GitHub để tải code từ repo về máy ảo Ubuntu.

    # BƯỚC 2: CÀI ĐẶT JAVA
    - name: Set up JDK 17
      id: setup-jdk            # Đặt ID.
      uses: actions/setup-java@v3 # Action cài đặt Java.
      with:
        java-version: '17'     # Chọn phiên bản Java 17.
        distribution: 'temurin' # Chọn nhà cung cấp JDK (Eclipse Temurin).
        # Mục đích: Chuẩn bị môi trường nếu cần chạy test hoặc build Maven bên ngoài Docker.

    # BƯỚC 3: ĐĂNG NHẬP AWS
    - name: Configure AWS Credentials
      id: aws-creds            # Đặt ID.
      uses: aws-actions/configure-aws-credentials@v1 # Action đăng nhập AWS CLI.
      with:
        # Lấy thông tin nhạy cảm từ "Két sắt" (Settings > Secrets) của GitHub Repo.
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }} # Token tạm thời (Bắt buộc với AWS Academy/Learner Lab).
        aws-region: ${{ env.AWS_REGION }} # Lấy region từ biến env khai báo ở trên.

    # BƯỚC 4: ĐĂNG NHẬP ECR (Docker Registry)
    - name: Login to Amazon ECR
      id: login-ecr            # Đặt ID (Quan trọng: Bước sau sẽ cần output của bước này).
      uses: aws-actions/amazon-ecr-login@v1 # Action giúp Docker máy ảo đăng nhập vào AWS ECR.

    # BƯỚC 5: BUILD & PUSH DOCKER IMAGE (Trọng tâm)
    - name: Build, Tag, and Push Image
      id: build-image          # Đặt ID.
      env:
        # Lấy địa chỉ Registry từ output của bước login-ecr (ví dụ: 0311...dkr.ecr...).
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        # Lấy mã Hash của Commit hiện tại làm Tag (Ví dụ: a1b2c3d). Giúp định danh phiên bản code.
        IMAGE_TAG: ${{ github.sha }}
      run: |  # Bắt đầu chạy các lệnh Linux (Bash shell)
        cd ${{ env.WORKING_DIRECTORY }}  # Di chuyển vào thư mục code (./api-gateway).
        
        # Lệnh 1: Build Image. Gán 2 tag cùng lúc:
        # - Tag theo mã commit (để lưu trữ lịch sử).
        # - Tag 'latest' (để ECS luôn pull bản mới nhất này).
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
        
        # Lệnh 2: Đẩy Image có tag commit lên ECR.
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        
        # Lệnh 3: Đẩy Image tag latest lên ECR.
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

    # BƯỚC 6: CẬP NHẬT ECS (Triển khai)
    - name: Force ECS Update
      id: ecs-update           # Đặt ID.
      run: |
        # Lệnh AWS CLI để cập nhật Service.
        # --force-new-deployment: Ép ECS tắt các container cũ và bật container mới (pull image mới nhất về).
        aws ecs update-service --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --force-new-deployment

    # BƯỚC 7: THÔNG BÁO THÀNH CÔNG
    - name: Success Notification
      if: success()            # Điều kiện: Chỉ chạy dòng này nếu TẤT CẢ các bước trên đều OK (Xanh lá).
      run: echo "🎉 DEPLOY API GATEWAY THÀNH CÔNG!"

    # BƯỚC 8: XỬ LÝ LỖI (ERROR HANDLER)
    - name: ⛔ ERROR HANDLER
      if: failure()            # Điều kiện: Chỉ chạy dòng này nếu CÓ BẤT KỲ bước nào phía trên bị lỗi (Đỏ).
      run: |
        echo "🚨 LỖI DEPLOY GATEWAY!"
        # Kiểm tra từng bước xem bước nào chết để in ra thông báo tiếng Việt dễ hiểu.
        if [[ "${{ steps.checkout.outcome }}" == 'failure' ]]; then echo "❌ Lỗi: Checkout Code"; fi
        if [[ "${{ steps.aws-creds.outcome }}" == 'failure' ]]; then echo "❌ Lỗi: AWS Credentials (Key sai/hết hạn)"; fi
        if [[ "${{ steps.build-image.outcome }}" == 'failure' ]]; then echo "❌ Lỗi: Docker Build/Push (Code lỗi hoặc Dockerfile sai)"; fi
        if [[ "${{ steps.ecs-update.outcome }}" == 'failure' ]]; then echo "❌ Lỗi: Không tìm thấy ECS Service (Sai tên Cluster hoặc Service)"; fi
        exit 1 # Báo cho GitHub biết là quy trình này đã thất bại (hiện dấu X đỏ ngoài dashboard).
```

#### 2\. File: `.github/workflows/deploy-auth.yml`



```YAML
name: Deploy Auth Service 🛡️

on:
  push:
    branches: [ "main", "pre-production" ]
    paths:
      - 'auth-service/**'
      - '.github/workflows/deploy-auth.yml'

env:
  AWS_REGION: ap-southeast-1
  ECR_REPOSITORY: auth-service
  ECS_SERVICE: auth-service
  ECS_CLUSTER: SGUTodolist-Cluster
  WORKING_DIRECTORY: ./auth-service

jobs:
  deploy:
    name: Build & Deploy
    runs-on: ubuntu-latest
    environment: production

    steps:
    - name: Checkout Code
      id: checkout
      uses: actions/checkout@v3

    - name: Set up JDK 17
      id: setup-jdk
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: Configure AWS Credentials
      id: aws-creds
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, Tag, and Push Image
      id: build-image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        IMAGE_TAG: ${{ github.sha }}
      run: |
        cd ${{ env.WORKING_DIRECTORY }}
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

    - name: Force ECS Update
      id: ecs-update
      run: |
        aws ecs update-service --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --force-new-deployment

    - name: Success Notification
      if: success()
      run: echo "🎉 DEPLOY AUTH SERVICE THÀNH CÔNG!"

    - name: ⛔ ERROR HANDLER
      if: failure()
      run: |
        echo "🚨 LỖI DEPLOY AUTH!"
        if [[ "${{ steps.aws-creds.outcome }}" == 'failure' ]]; then echo "❌ Lỗi: AWS Credentials"; fi
        if [[ "${{ steps.build-image.outcome }}" == 'failure' ]]; then echo "❌ Lỗi: Docker Build/Push"; fi
        exit 1

```

#### 3\. File: `.github/workflows/deploy-user.yml`



```YAML
name: Deploy User Service 👤

on:
  push:
    branches: [ "main", "pre-production" ]
    paths:
      - 'user-service/**'
      - '.github/workflows/deploy-user.yml'

env:
  AWS_REGION: ap-southeast-1
  ECR_REPOSITORY: user-service
  ECS_SERVICE: user-service
  ECS_CLUSTER: SGUTodolist-Cluster
  WORKING_DIRECTORY: ./user-service

jobs:
  deploy:
    name: Build & Deploy
    runs-on: ubuntu-latest
    environment: production

    steps:
    - name: Checkout Code
      id: checkout
      uses: actions/checkout@v3

    - name: Set up JDK 17
      id: setup-jdk
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: Configure AWS Credentials
      id: aws-creds
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, Tag, and Push Image
      id: build-image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        IMAGE_TAG: ${{ github.sha }}
      run: |
        cd ${{ env.WORKING_DIRECTORY }}
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

    - name: Force ECS Update
      id: ecs-update
      run: |
        aws ecs update-service --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --force-new-deployment

    - name: Success Notification
      if: success()
      run: echo "🎉 DEPLOY USER SERVICE THÀNH CÔNG!"

    - name: ⛔ ERROR HANDLER
      if: failure()
      run: |
        echo "🚨 LỖI DEPLOY USER!"
        if [[ "${{ steps.aws-creds.outcome }}" == 'failure' ]]; then echo "❌ Lỗi: AWS Credentials"; fi
        if [[ "${{ steps.build-image.outcome }}" == 'failure' ]]; then echo "❌ Lỗi: Docker Build/Push"; fi
        exit 1

```

#### 4\. File: `.github/workflows/deploy-task.yml`



```YAML
name: Deploy Taskflow Service ✅

on:
  push:
    branches: [ "main", "pre-production" ]
    paths:
      - 'taskflow-service/**'
      - '.github/workflows/deploy-task.yml'

env:
  AWS_REGION: ap-southeast-1
  ECR_REPOSITORY: taskflow-service
  ECS_SERVICE: taskflow-service
  ECS_CLUSTER: SGUTodolist-Cluster
  WORKING_DIRECTORY: ./taskflow-service

jobs:
  deploy:
    name: Build & Deploy
    runs-on: ubuntu-latest
    environment: production

    steps:
    - name: Checkout Code
      id: checkout
      uses: actions/checkout@v3

    - name: Set up JDK 17
      id: setup-jdk
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: Configure AWS Credentials
      id: aws-creds
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, Tag, and Push Image
      id: build-image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        IMAGE_TAG: ${{ github.sha }}
      run: |
        cd ${{ env.WORKING_DIRECTORY }}
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

    - name: Force ECS Update
      id: ecs-update
      run: |
        aws ecs update-service --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --force-new-deployment

    - name: Success Notification
      if: success()
      run: echo "🎉 DEPLOY TASKFLOW SERVICE THÀNH CÔNG!"

    - name: ⛔ ERROR HANDLER
      if: failure()
      run: |
        echo "🚨 LỖI DEPLOY TASKFLOW!"
        if [[ "${{ steps.aws-creds.outcome }}" == 'failure' ]]; then echo "❌ Lỗi: AWS Credentials"; fi
        if [[ "${{ steps.build-image.outcome }}" == 'failure' ]]; then echo "❌ Lỗi: Docker Build/Push"; fi
        exit 1

```

#### 5\. File: `.github/workflows/deploy-noti.yml`



```YAML
name: Deploy Notification Service 🔔

on:
  push:
    branches: [ "main", "pre-production" ]
    paths:
      - 'notification-service/**'
      - '.github/workflows/deploy-noti.yml'

env:
  AWS_REGION: ap-southeast-1
  ECR_REPOSITORY: notification-service
  ECS_SERVICE: notification-service
  ECS_CLUSTER: SGUTodolist-Cluster
  WORKING_DIRECTORY: ./notification-service

jobs:
  deploy:
    name: Build & Deploy
    runs-on: ubuntu-latest
    environment: production

    steps:
    - name: Checkout Code
      id: checkout
      uses: actions/checkout@v3

    - name: Set up JDK 17
      id: setup-jdk
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: Configure AWS Credentials
      id: aws-creds
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, Tag, and Push Image
      id: build-image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        IMAGE_TAG: ${{ github.sha }}
      run: |
        cd ${{ env.WORKING_DIRECTORY }}
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

    - name: Force ECS Update
      id: ecs-update
      run: |
        aws ecs update-service --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --force-new-deployment

    - name: Success Notification
      if: success()
      run: echo "🎉 DEPLOY NOTIFICATION SERVICE THÀNH CÔNG!"

    - name: ⛔ ERROR HANDLER
      if: failure()
      run: |
        echo "🚨 LỖI DEPLOY NOTIFICATION!"
        if [[ "${{ steps.aws-creds.outcome }}" == 'failure' ]]; then echo "❌ Lỗi: AWS Credentials"; fi
        if [[ "${{ steps.build-image.outcome }}" == 'failure' ]]; then echo "❌ Lỗi: Docker Build/Push"; fi
        exit 1

```

#### 6\. File: `.github/workflows/deploy-model.yml`



```YAML
name: Deploy AI Model 🧠

on:
  push:
    branches: [ "main", "pre-production" ]
    paths:
      - 'model/**'
      - '.github/workflows/deploy-model.yml'

env:
  AWS_REGION: ap-southeast-1
  ECR_REPOSITORY: ai-model-service
  ECS_SERVICE: ai-model-service
  ECS_CLUSTER: SGUTodolist-Cluster
  WORKING_DIRECTORY: ./model    # Chú ý thư mục là 'model'

jobs:
  deploy:
    name: Build & Deploy
    runs-on: ubuntu-latest
    environment: production

    steps:
    - name: Checkout Code
      id: checkout
      uses: actions/checkout@v3

    # Python không cần cài JDK

    - name: Configure AWS Credentials
      id: aws-creds
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, Tag, and Push Image
      id: build-image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        IMAGE_TAG: ${{ github.sha }}
      run: |
        cd ${{ env.WORKING_DIRECTORY }}
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

    - name: Force ECS Update
      id: ecs-update
      run: |
        aws ecs update-service --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --force-new-deployment

    - name: Success Notification
      if: success()
      run: echo "🎉 DEPLOY AI MODEL THÀNH CÔNG!"

    - name: ⛔ ERROR HANDLER
      if: failure()
      run: |
        echo "🚨 LỖI DEPLOY AI MODEL!"
        if [[ "${{ steps.aws-creds.outcome }}" == 'failure' ]]; then echo "❌ Lỗi: AWS Credentials"; fi
        if [[ "${{ steps.build-image.outcome }}" == 'failure' ]]; then echo "❌ Lỗi: Docker Build/Push"; fi
        exit 1

```

* * * * *

### BƯỚC 4: KÍCH HOẠT (TRIGGER)

Mở Terminal tại thư mục dự án và chạy:



```Bash
git add .
git commit -m "Official CI/CD Setup: 6 Services"
git push origin pre-production

```

**Ngay sau đó:**

1.  Vào GitHub > Tab **Actions**.

2.  Ta sẽ thấy 6 workflows bắt đầu chạy.

3.  Đợi chúng chuyển xanh ✅.


---

### LUỒNG TỔNG QUAN (CI/CD WORKFLOW)

Đây là hành trình của một dòng code từ máy của Developer lên tới Server (AWS ECS) một cách tự động hoàn toàn:

1.  **Code Change:** Developer sửa code (ví dụ: sửa màu nút bấm, fix lỗi logic) trên máy cá nhân.

2.  **Git Push:** Developer đẩy code lên GitHub.

    -   *Nếu đẩy vào nhánh cá nhân (`minh`, `dat`...):* Hệ thống **KHÔNG** làm gì cả (Tiết kiệm tài nguyên).

    -   *Nếu merge vào nhánh `main` hoặc `pre-production`:* Hệ thống **TỰ ĐỘNG KÍCH HOẠT**.

3.  **GitHub Actions (The Robot):**

    -   Tự động bật một máy ảo Ubuntu sạch sẽ.

    -   Tải code mới nhất về.

    -   Đăng nhập vào AWS bằng "Chìa khóa bí mật" (Secrets).

    -   Đóng gói code thành **Docker Image**.

    -   Đẩy Image lên kho chứa **AWS ECR**.

4.  **Deploy (AWS ECS):**

    -   GitHub Actions ra lệnh cho **AWS ECS**: *"Này, có bản cập nhật mới, hãy thay thế các container cũ đi!"*.

    -   AWS ECS tự động tắt container cũ, tải image mới về và khởi động container mới.

5.  **Kết quả:** Người dùng truy cập website sẽ thấy ngay tính năng mới mà không cần ai phải SSH vào server gõ lệnh.

---

### HƯỚNG DẪN SỬ DỤNG (DÀNH CHO DEVELOPER)

Để đảm bảo hệ thống CI/CD hoạt động trơn tru và tránh xung đột, mỗi thành viên trong nhóm cần tuân thủ quy trình sau mỗi khi muốn cập nhật code:

#### 1\. Quy tắc Nhánh (Branching Strategy)

-   **`main`**: Nhánh CHÍNH THỨC. Code ở đây là bản chạy thật (Production). **CẤM push trực tiếp**, chỉ được Merge từ nhánh khác sang.

-   **`pre-production`**: Nhánh CHẠY THỬ. Đây là nơi test tích hợp. Khi merge vào đây, server test sẽ tự động deploy.

-   **`developer`, `minh`, `dat`...**: Nhánh CÁ NHÂN. Thoải mái code, commit, push mà không sợ làm hỏng server.

#### 2\. Các bước Update Code (Quy trình chuẩn)

**Bước 1: Code và Test trên máy (Local)**

-   Code tính năng mới trên nhánh cá nhân (ví dụ `dat`).

-   Chạy thử ở local đảm bảo không lỗi.

**Bước 2: Đẩy code lên GitHub**

Bash

```
git add .
git commit -m "Thêm tính năng đăng nhập bằng Google"
git push origin dat  # Push lên nhánh của mình

```

*(Lúc này CI/CD chưa chạy, server vẫn yên bình).*

**Bước 3: Tạo Pull Request (PR) hoặc Merge**

-   Khi muốn đưa tính năng lên server cho cả nhóm test, hãy tạo **Pull Request** từ nhánh `dat` sang nhánh `pre-production`.

-   Hoặc nếu team nhỏ, có thể merge trực tiếp:

    Bash

    ```
    git checkout pre-production
    git pull origin pre-production  # Cập nhật code mới nhất từ server về trước
    git merge dat                   # Gộp code của mình vào
    git push origin pre-production  # Đẩy lên -> 🚀 BÙM! CI/CD KÍCH HOẠT

    ```

**Bước 4: Theo dõi và Tận hưởng**

-   Vào tab **Actions** trên GitHub Repo.

-   Bạn sẽ thấy workflow **"Deploy ... Service"** đang chạy (xoay vòng tròn vàng).

-   Chờ khoảng 3-5 phút đến khi hiện tích xanh ✅.

-   Vào web kiểm tra tính năng mới.

#### 3\. Xử lý khi gặp lỗi (Dấu ❌ Đỏ)

Nếu Actions báo đỏ (Failed):

1.  Bấm vào workflow bị lỗi.

2.  Bấm vào mục **Build & Deploy**.

3.  Kéo xuống dưới cùng, tìm phần **⛔ ERROR HANDLER**.

4.  Đọc dòng thông báo tiếng Việt (ví dụ: *"❌ Lỗi tại bước [Docker Build]: Code lỗi..."*) để biết nguyên nhân và sửa lại code.

* * * * *

**Tóm lại:**

> **"Code ở nhánh con -> Merge vào `pre-production` -> Tự động lên Server Test -> Test ngon -> Merge vào `main` -> Tự động lên Server Thật."**