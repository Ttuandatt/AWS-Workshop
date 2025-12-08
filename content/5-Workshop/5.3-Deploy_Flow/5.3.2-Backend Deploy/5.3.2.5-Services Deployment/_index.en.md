+++
title = "Services Deployment "
weight = 5
chapter = false
pre = " <b> 5.3.2.5. </b> "
alwaysopen = true
+++

This phase deploys containerized applications to ECS Fargate following a specific order to ensure proper dependency resolution.

### Critical Network Configuration

**For ALL service deployments, the following network settings are mandatory:**

1\.  **VPC**: `SGU-Microservices-VPC`

2\.  **Subnets**: Select both Public Subnets (required for image pulling)

3\.  **Security Group**: `ecs-app-sg` (remove default)

4\.  **Public IP**: **Turned on** (critical for ECR access)

* * * * *

### Deployment Priority Order

Services must be deployed in the following sequence to respect dependencies:

**Group 1: Business Logic Services** (Deploy first - required by API Gateway)

1\.  Auth Service

2\.  User Service

3\.  Taskflow Service

4\.  Notification Service

**Group 2: Internal Services** (Deploy second) 5. AI Model Service

**Group 3: Entry Point** (Deploy last) 6. API Gateway

* * * * *

### Group 1: Business Logic Services Deployment

These services require both ALB integration (for external access) and Service Discovery (for internal communication).

**Deployment Parameters Reference Table:**

| Service | Task Definition | Service Name | Target Group | Service Discovery Name |

| --- | --- | --- | --- | --- |

| Auth | `auth-service-td` | `auth-service` | `auth-tg` | `auth` |

| User | `user-service-td` | `user-service` | `user-tg` | `user` |

| Taskflow | `taskflow-service-td` | `taskflow-service` | `task-tg` | `taskflow` |

| Notification | `notification-service-td` | `notification-service` | `noti-tg` | `notification` |

**Standard Deployment Process (Repeat 4 times):**

1\.  Navigate to **ECS Cluster** → **Services** → **Create**

2\.  **Service Configuration:**

    -   **Task definition family**: Select corresponding task definition (e.g., `auth-service-td`)

    -   **Revision**: Latest

    -   **Service name**: Enter service name (e.g., `auth-service`)

3\.  **Compute Configuration:**

    -   **Compute options**: Capacity provider strategy

    -   **Capacity provider strategy**: Use custom (Advanced)

        -   **Capacity provider**: FARGATE

        -   **Base**: 0

        -   **Weight**: 1

    -   **Platform version**: LATEST

4\.  **Deployment Configuration:**

    -   **Application type**: Service

    -   **Scheduling strategy**: Replica

    -   **Desired tasks**: 1

    -   **Desired tasks**: 1

    -       
5\.  **Networking:**

    -   **VPC**: `SGU-Microservices-VPC`

    -   **Subnets**: Select both Public Subnets

        -   `SGU-Microservices-subnet-public1-ap-southeast-1a`

        -   `SGU-Microservices-subnet-public2-ap-southeast-1b`

    -   **Security group**: Remove default, select `ecs-app-sg`

    -   **Public IP**: Turned on

6\.  **Service Connect:**

    -   **Do not enable** (using traditional DNS-based Service Discovery)

7\.  **Service Discovery:**

    -   **Enable**: Use service discovery

    -   **Configure namespace**: Select an existing namespace

    -   **Namespace**: `sgu.local`

    -   **Configure service discovery service**: `Select an existing service discovery service`

    -   **Existing service discovery service**: Choose discovery name (e.g., `auth`)

    -   **Result DNS**: `auth.sgu.local` (for example)

8\.  **Load Balancing:**

    -   **Enable**: Use load balancing

    -   **Load balancer type**: Application Load Balancer

    -   **Load balancer**: `sgu-alb`

    -   **Container to load balance**: Select service container (e.g., `auth-service 9999:9999`)

    -   **Listener**: Use an existing listener → HTTPS:443

    -   **Target group**: Use an existing target group → Select corresponding TG (e.g., `auth-tg`)

9\.  **Service Auto Scaling:**

    -   **Disable** (for cost optimization)

10\. Click **Create**

**Wait for each service to reach RUNNING state before deploying the next one.**

* * * * *

### Group 2: AI Model Service Deployment

This is an internal service accessed only through API Gateway.

**Configuration:**

1\.  Navigate to **ECS Cluster** → **Services** → **Create**

2\.  **Service Configuration:**

    -   **Task definition**: `ai-model-service-td`

    -   **Service name**: `ai-model-service`

    -   **Desired tasks**: 1

3\.  **Networking:**

    -   **VPC**: `SGU-Microservices-VPC`

    -   **Subnets**: Both Public Subnets

    -   **Security group**: `ecs-app-sg`

    -   **Public IP**: Turned on

4\.  **Load Balancing:**

    -   **Disable** (internal service only)

5\.  **Service Discovery:**

    -   **Enable**: Use service discovery

    -   **Namespace**: `sgu.local`

    -   **Service discovery name**: `ai-model`

    -   **Result DNS**: `ai-model.sgu.local`

6\.  Click **Create**

* * * * *

### Group 3: API Gateway Deployment

This is the main entry point that routes traffic to all backend services.

**Configuration:**

1\.  Navigate to **ECS Cluster** → **Services** → **Create**

2\.  **Service Configuration:**

    -   **Task definition**: `api-gateway-td`

    -   **Service name**: `api-gateway`

    -   **Desired tasks**: 1

3\.  **Compute Configuration:**

    -   Same as Group 1 services

4\.  **Networking:**

    -   **VPC**: `SGU-Microservices-VPC`

    -   **Subnets**: Both Public Subnets

    -   **Security group**: `ecs-app-sg`

    -   **Public IP**: Turned on

5\.  **Service Discovery:**

    -   **Enable**: Use service discovery

    -   **Namespace**: `sgu.local`

    -   **Service discovery name**: `api-gateway`

6\.  **Load Balancing:**

    -   **Enable**: Use load balancing

    -   **Load balancer type**: Application Load Balancer

    -   **Load balancer**: `sgu-alb`

    -   **Container to load balance**: `api-gateway:8080`

    -   **Listener**: Use an existing listener → **HTTP:80**

    -   **Target group**: Use an existing target group → `api-gateway-tg`

7\.  Click **Create**

* * * * *

### Deployment Monitoring and Troubleshooting

After deploying all services, monitor their status in the ECS Cluster.

**Expected Task States:**

Navigate to **Cluster** → **Tasks** tab

1\.  **Normal progression**: `PROVISIONING` → `PENDING` → `RUNNING`

2\.  **PENDING state**: 1-2 minutes is normal (downloading image from ECR)

3\.  **RUNNING state**: Task is healthy and serving traffic

**Common Issues and Resolution:**

#### Issue 1: Task Status = STOPPED

**Symptom**: Task stops immediately after starting

**Diagnosis Steps:**

1\.  Click on the stopped task ID

2\.  Check **Stopped reason** field

3\.  Switch to **Logs** tab for detailed error messages

**Common Causes:**

| Error Message | Cause | Solution |

| --- | --- | --- |

| `Connection refused` | Incorrect RDS/Redis endpoint in environment variables | Verify endpoint values in Task Definition |

| `CannotPullContainerError` | Missing Public IP or subnet without internet access | Ensure Public IP is enabled and using Public Subnets |

| `Health check failed` | Service startup time exceeds ALB timeout | Increase health check interval and timeout in Target Group settings |

| `ResourceInitializationError` | Insufficient CPU/Memory | Verify task size configuration |

**Solution for Health Check Failures:**

1\.  Navigate to **Target Group** → Select the failing TG

2\.  **Health checks** tab → Edit

3\.  Increase values:

    -   **Interval**: 60 seconds

    -   **Timeout**: 30 seconds

    -   **Healthy threshold**: 3

4\.  Save changes

#### Issue 2: Container Keeps Restarting

**Possible Causes:**

-   Database connection failures

-   Missing environment variables

-   Application configuration errors

**Diagnosis:**

1\.  Check CloudWatch Logs for the task

2\.  Verify all environment variables are correctly set

3\.  Confirm RDS and Redis are accessible from ECS security group

* * * * *

### Deployment Verification Checklist

Before proceeding to final configuration:

**Services Status:**

-   [ ]  Auth Service: RUNNING (1/1 tasks)

-   [ ]  User Service: RUNNING (1/1 tasks)

-   [ ]  Taskflow Service: RUNNING (1/1 tasks)

-   [ ]  Notification Service: RUNNING (1/1 tasks)

-   [ ]  AI Model Service: RUNNING (1/1 tasks)

-   [ ]  API Gateway: RUNNING (1/1 tasks)

**Target Groups Health:**

-   [ ]  `auth-tg`: 1 healthy target

-   [ ]  `user-tg`: 1 healthy target

-   [ ]  `task-tg`: 1 healthy target

-   [ ]  `noti-tg`: 1 healthy target

-   [ ]  `api-gateway-tg`: 1 healthy target

**Service Discovery:**

-   [ ]  All services registered in Cloud Map

-   [ ]  DNS names resolving correctly (verify in Route 53)

* * * * *

<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.4-Task Definitions Creation (Configure settings, FIX environment variables)" %}}" style="text-decoration: none; font-weight: bold;">
⬅ BƯỚC 4: Task Definitions Creation
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.6-Completion & Verification (Route 53, Google Console, Final Test)" %}}" style="text-decoration: none; font-weight: bold;">
BƯỚC 6: Completion & Verification ➡
</a>
</div>