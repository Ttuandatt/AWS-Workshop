+++
title = "Code Update & Image Build (Create new Docker Image)"
weight = 3
chapter = false
pre = " <b> 5.3.2.3. </b> "
alwaysopen = true
+++

This phase focuses on adapting the application source code for the cloud environment. We will configure Cross-Origin Resource Sharing (CORS) for the reactive API Gateway, refactor application configurations to use dynamic environment variables, and finally build and push all microservice Docker images to the Amazon Elastic Container Registry (ECR).

**Objectives:**

1.  Configure the code to work seamlessly in both Local (Docker Compose) and Cloud (AWS ECS) environments.

2.  Create Repositories on AWS ECR.

3.  Build and Push all 6 Docker images to AWS.

* * * * *

### 1. CREATE REPOSITORIES ON AWS ECR

Before pushing images, you must create a "repository" for each service. Run the following commands in your Terminal (PowerShell or Git Bash):


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

* * * * *

### 2. BUILD AND PUSH IMAGES

Execute these commands from the **root directory** of your project (`todolist-backend`).

**1\. Login Docker to AWS:** *(Replace `031133710884` with your AWS Account ID if it's different)*



```bash
aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com

```

**2\. Build & Push each of the 6 Services:**

**Service 1: API Gateway**


```bash
cd api-gateway
docker build -t api-gateway:latest .
docker tag api-gateway:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/api-gateway:latest
docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/api-gateway:latest
cd ..

```

**Service 2: Auth Service**

```bash
cd auth-service
docker build -t auth-service:latest .
docker tag auth-service:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/auth-service:latest
docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/auth-service:latest
cd ..

```

**Service 3: User Service**

```bash
cd user-service
docker build -t user-service:latest .
docker tag user-service:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/user-service:latest
docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/user-service:latest
cd ..

```

**Service 4: Taskflow Service**

```bash
cd taskflow-service
docker build -t taskflow-service:latest .
docker tag taskflow-service:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/taskflow-service:latest
docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/taskflow-service:latest
cd ..

```

**Service 5: Notification Service**

```bash

cd notification-service
docker build -t notification-service:latest .
docker tag notification-service:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/notification-service:latest
docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/notification-service:latest
cd ..

```

**Service 6: AI Model Service** *(Note: The folder name is `model`)*

```bash

cd model
docker build -t ai-model-service:latest .
docker tag ai-model-service:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/ai-model-service:latest
docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/ai-model-service:latest
cd ..

```

* * * * *

### EXPECTED RESULT

After running all the commands, you can verify by running:

```bash

aws ecr describe-repositories --region ap-southeast-1

```

If you see a list of 6 repositories and no errors during the push process, you have successfully completed this phase.

You can now proceed to creating the **Task Definitions**.
* * * * *

<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.2-Infrastructure & ALB Setup (RDS, Redis, Cloud Map, ALB Routing)" %}}" style="text-decoration: none; font-weight: bold;">
⬅ STEP 2: Infrastructure & ALB Setup
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.4-Task Definitions Creation (Configure settings, FIX environment variables)" %}}" style="text-decoration: none; font-weight: bold;">
STEP 4: Task Definitions Creation ➡
</a>
</div>