+++
title = "Task Definitions Creation (Configure settings, Fix environment variables)"
weight = 4
chapter = false
pre = " <b> 5.3.2.4. </b> "
alwaysopen = true
+++

Task Definitions serve as blueprints that define how ECS should run containers, including resource allocation and environment configuration.

### Preparation

**Record the following values before creating Task Definitions:**

1.  **RDS Endpoint**: `sgu-todolist-db.[random].ap-southeast-1.rds.amazonaws.com`
2.  **Redis Endpoint**: `sgu-redis.[random].cache.amazonaws.com` (without :6379)
3.  **Kafka Bootstrap Server**: `kafka.sgu.local:9092`
4.  **Google OAuth Credentials**:
    -   Client ID
    -   Client Secret
5.  **ECR Image URIs**: All 6 service images from ECR

* * * * *

### Standard Task Definition Process

For each service, follow this standard configuration process:

**Base Configuration (Common for all services):**

1.  Navigate to **Amazon ECS** → **Task definitions** → **Create new task definition**
2.  Infrastructure configuration:

| Parameter | Value |
| --- | --- |
| Launch type | AWS Fargate |
| Operating system | Linux/X86_64 |
| CPU | 0.5 vCPU |
| Memory | 1 GB |
| Task execution role | `ecsTaskExecutionRole` |

1.  Container configuration:
    -   **Protocol**: TCP
    -   **App protocol**: HTTP
    -   **Logging**: Enable "Use log collection"

* * * * *

### Task Definition 1: Auth Service

| Parameter | Value |
| --- | --- |
| Task definition family | `auth-service-td` |
| Container name | `auth-service` |
| Image URI | `[ECR-URI]/auth-service:latest` |
| Container port | 9999 |

**Environment Variables:**

| Key | Value |
| --- | --- |
| `SPRING_DATASOURCE_URL` | `jdbc:mysql://[RDS-ENDPOINT]:3306/aws_todolist_database?allowPublicKeyRetrieval=true&useSSL=false&serverTimezone=UTC` |
| `SPRING_DATASOURCE_USERNAME` | `root` |
| `SPRING_DATASOURCE_PASSWORD` | `[your-password]` |
| `SPRING_DATA_REDIS_HOST` | `[REDIS-ENDPOINT]` |
| `SPRING_DATA_REDIS_PORT` | `6379` |
| `SPRING_KAFKA_BOOTSTRAP_SERVERS` | `kafka.sgu.local:9092` |
| `SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_CLIENT_ID` | `[GOOGLE-CLIENT-ID]` |
| `SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_CLIENT_SECRET` | `[GOOGLE-CLIENT-SECRET]` |
| `SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_REDIRECT_URI` | `https://api.sgutodolist.com/api/auth/login/oauth2/code/google` |
| `APP_OAUTH2_REDIRECT_URI` | `https://sgutodolist.com/oauth2/redirect` |

* * * * *

### Task Definition 2: User Service

| Parameter | Value |
| --- | --- |
| Task definition family | `user-service-td` |
| Container name | `user-service` |
| Image URI | `[ECR-URI]/user-service:latest` |
| Container port | 8081 |

**Environment Variables:**

| Key | Value |
| --- | --- |
| `SPRING_DATASOURCE_URL` | Same as Auth Service |
| `SPRING_DATASOURCE_USERNAME` | `root` |
| `SPRING_DATASOURCE_PASSWORD` | `[your-password]` |
| `SPRING_DATA_REDIS_HOST` | `[REDIS-ENDPOINT]` |
| `SPRING_DATA_REDIS_PORT` | `6379` |
| `SPRING_KAFKA_BOOTSTRAP_SERVERS` | `kafka.sgu.local:9092` |

* * * * *

### Task Definition 3: Taskflow Service

| Parameter | Value |
| --- | --- |
| Task definition family | `taskflow-service-td` |
| Container name | `taskflow-service` |
| Image URI | `[ECR-URI]/taskflow-service:latest` |
| Container port | 8082 |

**Environment Variables:**

| Key | Value |
| --- | --- |
| `SPRING_DATASOURCE_URL` | Same as Auth Service |
| `SPRING_DATASOURCE_USERNAME` | `root` |
| `SPRING_DATASOURCE_PASSWORD` | `[your-password]` |
| `SPRING_DATA_REDIS_HOST` | `[REDIS-ENDPOINT]` |
| `SPRING_DATA_REDIS_PORT` | `6379` |
| `SPRING_KAFKA_BOOTSTRAP_SERVERS` | `kafka.sgu.local:9092` |

* * * * *

### Task Definition 4: Notification Service

| Parameter | Value |
| --- | --- |
| Task definition family | `notification-service-td` |
| Container name | `notification-service` |
| Image URI | `[ECR-URI]/notification-service:latest` |
| Container port | 9998 |

**Environment Variables:**

| Key | Value |
| --- | --- |
| `SPRING_DATASOURCE_URL` | Same as Auth Service |
| `SPRING_DATASOURCE_USERNAME` | `root` |
| `SPRING_DATASOURCE_PASSWORD` | `[your-password]` |
| `SPRING_DATA_REDIS_HOST` | `[REDIS-ENDPOINT]` |
| `SPRING_DATA_REDIS_PORT` | `6379` |
| `SPRING_KAFKA_BOOTSTRAP_SERVERS` | `kafka.sgu.local:9092` |
| `CLIENT_URL` | `https://sgutodolist.com` |

* * * * *

### Task Definition 5: AI Model Service

| Parameter | Value |
| --- | --- |
| Task definition family | `ai-model-service-td` |
| Container name | `ai-model-service` |
| Image URI | `[ECR-URI]/ai-model-service:latest` |
| Container port | 9997 |

**Environment Variables:**

If your Python Flask AI service requires database connectivity, add the same database variables as other services. Otherwise, leave environment variables empty.

* * * * *

### Task Definition 6: API Gateway

**Critical Configuration**: This service orchestrates all backend services and must know their internal addresses.

| Parameter | Value |
| --- | --- |
| Task definition family | `api-gateway-td` |
| Container name | `api-gateway` |
| Image URI | `[ECR-URI]/api-gateway:latest` |
| Container port | 8080 |

**Environment Variables:**

| Key | Value | Purpose |
| --- | --- | --- |
| `SPRING_DATA_REDIS_HOST` | `[REDIS-ENDPOINT]` | Rate limiting |
| `SPRING_DATA_REDIS_PORT` | `6379` | Redis connection |
| `AUTH_SERVICE_URL` | `http://auth.sgu.local:9999` | Internal routing |
| `USER_SERVICE_URL` | `http://user.sgu.local:8081` | Internal routing |
| `TASKFLOW_SERVICE_URL` | `http://taskflow.sgu.local:8082` | Internal routing |
| `NOTIFICATION_SERVICE_URL` | `http://notification.sgu.local:9998` | Internal routing |
| `AI_MODEL_SERVICE` | `http://ai-model.sgu.local:9997` | Internal routing |
| `CORS_ALLOWED_ORIGINS` | `https://sgutodolist.com` | Frontend access |

* * * * *

### Task Definition Validation

After creating all 6 Task Definitions, verify:

-   [ ]  All Task Definitions created successfully
-   [ ]  Latest revisions available for each family
-   [ ]  Environment variables correctly configured
-   [ ]  Image URIs point to correct ECR repositories
-   [ ]  Logging enabled for all containers

* * * * *

<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.3-Code Update & Image Build (Create new Docker Image)" %}}" style="text-decoration: none; font-weight: bold;">
⬅ BƯỚC 3: Code Update & Image Build
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.5-Services Deployment (Following priority order and verification)" %}}" style="text-decoration: none; font-weight: bold;">
BƯỚC 5: Services Deployment ➡
</a>
</div>