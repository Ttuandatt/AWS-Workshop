+++
title = "Task Definitions Creation"
weight = 4
chapter = false
pre = " <b> 5.3.2.4. </b> "
alwaysopen = true
+++


Task Definitions serve as blueprints that define how ECS should run containers, including resource allocation (RAM/CPU) and essential environment variables for service connectivity.

### Preparation

**Record the following values before starting:**

1.  **RDS Endpoint**: `sgu-todolist-db.[random].ap-southeast-1.rds.amazonaws.com`

2.  **Redis Endpoint**: `sgu-redis.[random].cache.amazonaws.com` (Without :6379)

3.  **ECR Image URIs**: The URIs of the 6 repositories from the Image Build step.

4.  **Google OAuth Credentials**: Client ID and Client Secret.

* * * * *

### Standard Task Definition Process

For each service, follow this standard configuration process:

**1\. Base Configuration:**

-   Navigate to **Amazon ECS** → **Task definitions** → **Create new task definition**.

-   **Infrastructure:**

    -   Launch type: **AWS Fargate**

    -   Operating system: **Linux/X86_64**

    -   Task Execution Role: `ecsTaskExecutionRole`

-   **Container Details:**

    -   Protocol: **TCP**

    -   Port Mapping: Refer to the specific service details below.

    -   Log collection: **Turned on** (Essential for debugging).

* * * * *

### 1\. Kafka Server (Infrastructure Backbone)

*Note: Kafka must be deployed first so other services can connect upon startup.*

| **Parameter** | **Value** |
| --- | --- |
| **Family Name** | `kafka-server-td` |
| **Task Memory** | **2 GB** (Critical) |
| **Task CPU** | **1 vCPU** |
| **Container Port** | `9092` |

**Environment Variables (Copy exactly):**

| **Key** | **Value** |
| --- | --- |
| `ALLOW_PLAINTEXT_LISTENER` | `yes` |
| `KAFKA_CFG_ADVERTISED_LISTENERS` | `PLAINTEXT://kafka.sgu.local:9092` |
| `KAFKA_CFG_CONTROLLER_LISTENER_NAMES` | `CONTROLLER` |
| `KAFKA_CFG_CONTROLLER_QUORUM_VOTERS` | `0@127.0.0.1:9093` |
| `KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP` | `CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT` |
| `KAFKA_CFG_LISTENERS` | `PLAINTEXT://:9092,CONTROLLER://:9093` |
| `KAFKA_CFG_LOG_DIRS` | `/tmp/kafka-logs` |
| `KAFKA_CFG_NODE_ID` | `0` |
| `KAFKA_CFG_PROCESS_ROLES` | `controller,broker` |
| `KAFKA_HEAP_OPTS` | `-Xmx512m` |

* * * * *

### 2\. Auth Service (Critical Security Service)

*Note: This service requires high RAM to handle Login processes and Encryption.*

| **Parameter** | **Value** |
| --- | --- |
| **Family Name** | `auth-service-td` |
| **Task Memory** | **2 GB** (Mandatory to prevent Exit Code 137) |
| **Task CPU** | **0.5 vCPU** |
| **Container Port** | `9999` |

**Environment Variables:**

| **Key** | **Value** | **Notes** |
| --- | --- | --- |
| `SERVER_PORT` | `9999` |  |
| `JAVA_OPTS` | `-Xmx768m` | Must include the hyphen `-`. |
| `SPRING_JPA_HIBERNATE_DDL_AUTO` | **`none`** | **CRITICAL:** Prevents 500 Errors caused by DB schema conflicts. |
| `SERVER_FORWARD_HEADERS_STRATEGY` | `native` | Fixes HTTP/HTTPS redirect issues behind ALB. |
| `SPRING_DATASOURCE_URL` | `jdbc:mysql://[RDS-ENDPOINT]:3306/aws_todolist_database?allowPublicKeyRetrieval=true&useSSL=false&serverTimezone=UTC` |  |
| `SPRING_DATASOURCE_USERNAME` | `root` |  |
| `SPRING_DATASOURCE_PASSWORD` | `[DB_PASSWORD]` |  |
| `SPRING_DATASOURCE_HIKARI_MAXIMUM_POOL_SIZE` | `5` |  |
| `SPRING_DATA_REDIS_HOST` | `[REDIS-ENDPOINT]` |  |
| `SPRING_DATA_REDIS_PORT` | `6379` |  |
| `SPRING_DATA_REDIS_SSL_ENABLED` | **`true`** | Mandatory for AWS ElastiCache. |
| `SPRING_KAFKA_BOOTSTRAP_SERVERS` | `kafka.sgu.local:9092` |  |
| `DOMAIN_FRONTEND` | `https://sgutodolist.com` |  |
| `APP_OAUTH2_REDIRECT_URI` | `https://sgutodolist.com/oauth2/redirect` |  |
| `SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_CLIENT_ID` | `[GOOGLE_CLIENT_ID]` |  |
| `SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_CLIENT_SECRET` | `[GOOGLE_CLIENT_SECRET]` |  |
| `SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_REDIRECT_URI` | `https://sgutodolist.com/api/auth/login/oauth2/code/google` | Note: Include `/api/auth` in path. |

* * * * *

### 3\. User Service & Taskflow Service

*These are business logic services requiring permission to create DB tables on first run.*

| **Parameter** | **Value** |
| --- | --- |
| **Family Name** | `user-service-td` / `taskflow-service-td` |
| **Task Memory** | **1 GB** |
| **Task CPU** | **0.5 vCPU** |
| **Container Port** | `8081` (User) / `8082` (Taskflow) |

**Environment Variables (Common for both):**

| **Key** | **Value** | **Notes** |
| --- | --- | --- |
| `SERVER_PORT` | `8081` (User) or `8082` (Taskflow) |  |
| `JAVA_OPTS` | `-Xmx512m` |  |
| `SPRING_JPA_HIBERNATE_DDL_AUTO` | **`update`** | Allows service to create tables on startup. |
| `SPRING_DATASOURCE_URL` | *(Same as Auth Service)* |  |
| `SPRING_DATASOURCE_USERNAME` | `root` |  |
| `SPRING_DATASOURCE_PASSWORD` | `[DB_PASSWORD]` |  |
| `SPRING_DATASOURCE_HIKARI_MAXIMUM_POOL_SIZE` | `5` |  |
| `SPRING_DATA_REDIS_HOST` | `[REDIS-ENDPOINT]` |  |
| `SPRING_DATA_REDIS_PORT` | `6379` |  |
| `SPRING_DATA_REDIS_SSL_ENABLED` | `true` |  |
| `SPRING_KAFKA_BOOTSTRAP_SERVERS` | `kafka.sgu.local:9092` |  |

* * * * *

### 4\. API Gateway (The Orchestrator)

*The most critical service for internal routing.*

| **Parameter** | **Value** |
| --- | --- |
| **Family Name** | `api-gateway-td` |
| **Task Memory** | **1 GB** |
| **Task CPU** | **0.5 vCPU** |
| **Container Port** | `8080` |

**Environment Variables:**

| **Key** | **Value** | **Explanation** |
| --- | --- | --- |
| `SERVER_PORT` | `8080` |  |
| `CORS_ALLOWED_ORIGINS` | `https://sgutodolist.com,https://www.sgutodolist.com` | Allows Frontend API access. |
| `AUTH_SERVICE_URL` | `http://auth.sgu.local:9999` | Internal routing via Service Discovery. |
| `USER_SERVICE_URL` | **`http://user.sgu.local:8081`** | Note the name: `user` (not `user-service`). |
| `TASKFLOW_SERVICE_URL` | **`http://taskflow.sgu.local:8082`** | Note the name: `taskflow`. |
| `NOTIFICATION_SERVICE_URL` | **`http://notification.sgu.local:9998`** | Note the name: `notification`. |
| `SPRING_DATA_REDIS_HOST` | `[REDIS-ENDPOINT]` | Used for Rate Limiting. |
| `SPRING_DATA_REDIS_PORT` | `6379` |  |
| `SPRING_DATA_REDIS_SSL_ENABLED` | `true` |  |

* * * * *

### Task Definition Validation Checklist

After creation, please verify:

-   [ ] **RAM Allocation:** Auth Service must be **2GB**, others at least **1GB**.

-   [ ] **Redis SSL:** Variable `SPRING_DATA_REDIS_SSL_ENABLED` = `true` is set for all services connecting to Redis.

-   [ ] **Internal URLs:** All `_SERVICE_URL` variables in Gateway must end with `.sgu.local`.

-   [ ] **DDL Auto:** Auth Service must be `none`, User/Taskflow should be `update`.

<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.3-Code Update & Image Build (Create new Docker Image)" %}}" style="text-decoration: none; font-weight: bold;">
⬅ STEP 3: Code Update & Image Build
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.5-Services Deployment" %}}" style="text-decoration: none; font-weight: bold;">
STEP 5: Services Deployment ➡
</a>
</div>