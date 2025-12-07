+++
title = "Code Update & Image Build (Create new Docker Image)"
weight = 3
chapter = false
pre = " <b> 5.3.2.3. </b> "
alwaysopen = true
+++

This phase prepares the application code and builds Docker images for deployment to ECR.

### API Gateway Configuration

The API Gateway requires specific configuration changes to support the AWS deployment environment.

#### CORS Configuration File

**Purpose**: Handle Cross-Origin Resource Sharing for the React frontend in a reactive environment.

**File Location**: `api-gateway/src/main/java/aws/todolist/api_gateway/config/CorsConfig.java`

**Content:**


```java

package aws.todolist.api_gateway.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.reactive.CorsWebFilter;
import org.springframework.web.cors.reactive.UrlBasedCorsConfigurationSource;
import java.util.Arrays;

@Configuration
public class CorsConfig {

    @Bean
    public CorsWebFilter corsWebFilter() {
        CorsConfiguration corsConfig = new CorsConfiguration();

        // Allowed origins
        corsConfig.setAllowedOrigins(Arrays.asList(
            "https://sgutodolist.com",
            "https://www.sgutodolist.com",
            "http://localhost:3000", // Development
            "http://localhost:4200", // Development (backup)
            "http://localhost:5500"
        ));

        // Configuration
        corsConfig.setMaxAge(3600L);
        corsConfig.addAllowedMethod("*");
        corsConfig.addAllowedHeader("*");
        corsConfig.setAllowCredentials(true);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", corsConfig);

        return new CorsWebFilter(source);
    }
}
```

* * * * *

#### Application Configuration Update

**Purpose**: Enable environment variable-based configuration for dynamic service discovery.

**File Location**: `api-gateway/src/main/resources/application.yml`

**Content:**


```yaml
server:
  port: 8080

jwt:
  secret: 843567893696976453275974432697R634976R738467TR678T34865R6834R8763T478378637664538745673865783678548735687R3

spring:
  application:
    name: api-gateway

  cloud:
    gateway:
      routes:
        - id: ai-model-service
          uri: ${AI_MODEL_SERVICE:http://localhost:9997}
          predicates:
            - Path=/api/model/**

        - id: user-service
          uri: ${USER_SERVICE_URL:http://localhost:8081}
          predicates:
            - Path=/api/user/**

        - id: taskflow-service
          uri: ${TASKFLOW_SERVICE_URL:http://localhost:8082}
          predicates:
            - Path=/api/taskflow/**

        - id: auth-service
          uri: ${AUTH_SERVICE_URL:http://localhost:9999}
          predicates:
            - Path=/api/auth/**

        - id: notification-ws
          uri: ${NOTIFICATION_SERVICE_URL:http://localhost:9998}
          predicates:
            - Path=/api/notification/ws/**

        - id: notification-rest
          uri: ${NOTIFICATION_SERVICE_URL:http://localhost:9998}
          predicates:
            - Path=/api/notification/**

  data:
    redis:
      host: ${SPRING_DATA_REDIS_HOST:redis}
      port: ${SPRING_DATA_REDIS_PORT:6379}
      timeout: 6000
      lettuce:
        pool:
          max-active: 8
          max-idle: 8
          min-idle: 0
          max-wait: -1ms

management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    health:
      show-details: always

springdoc:
  api-docs:
    enabled: true
  swagger-ui:
    urls:
      - name: user-service
        url: ${USER_SERVICE_URL:http://localhost:8081}/api/user/v3/api-docs
      - name: taskflow-service
        url: ${TASKFLOW_SERVICE_URL:http://localhost:8082}/api/taskflow/v3/api-docs
      - name: auth-service
        url: ${AUTH_SERVICE_URL:http://localhost:9999}/api/auth/v3/api-docs
      - name: notification-service
        url: ${NOTIFICATION_SERVICE_URL:http://localhost:9998}/api/notification/v3/api-docs
```

* * * * *

### Docker Image Build and Push

**Prerequisites:**

-   Docker Desktop running
-   AWS CLI configured
-   Terminal opened at project root directory (`todolist-backend`)

**Step 1: Authenticate with ECR**


```bash
aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com
```

**Step 2: Build and Push Images**

Execute the following commands sequentially for each service:

#### Service 1: API Gateway



```bash
cd api-gateway
docker build -t api-gateway:latest .
docker tag api-gateway:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/api-gateway:latest
docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/api-gateway:latest
cd ..
```

#### Service 2: Auth Service



```bash
cd auth-service
docker build -t auth-service:latest .
docker tag auth-service:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/auth-service:latest
docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/auth-service:latest
cd ..
```

#### Service 3: User Service


```bash

cd user-service
docker build -t user-service:latest .
docker tag user-service:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/user-service:latest
docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/user-service:latest
cd ..
```

#### Service 4: Taskflow Service



```bash
cd taskflow-service
docker build -t taskflow-service:latest .
docker tag taskflow-service:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/taskflow-service:latest
docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/taskflow-service:latest
cd ..
```

#### Service 5: Notification Service



```bash
cd notification-service
docker build -t notification-service:latest .
docker tag notification-service:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/notification-service:latest
docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/notification-service:latest
cd ..
```

#### Service 6: AI Model Service



```bash
cd model
docker build -t ai-model-service:latest .
docker tag ai-model-service:latest 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/ai-model-service:latest
docker push 031133710884.dkr.ecr.ap-southeast-1.amazonaws.com/ai-model-service:latest
cd ..
```

* * * * *

### Image Build Verification

After pushing all images, verify they are available in ECR:



```bash
aws ecr describe-images --repository-name api-gateway --region ap-southeast-1
aws ecr describe-images --repository-name auth-service --region ap-southeast-1
aws ecr describe-images --repository-name user-service --region ap-southeast-1
aws ecr describe-images --repository-name taskflow-service --region ap-southeast-1
aws ecr describe-images --repository-name notification-service --region ap-southeast-1
aws ecr describe-images --repository-name ai-model-service --region ap-southeast-1
```

**Checklist:**

-   [ ]  All 6 images successfully built without errors
-   [ ]  All 6 images successfully pushed to ECR
-   [ ]  Image URIs recorded for Task Definition configuration

* * * * *

<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.2-Infrastructure & ALB Setup (RDS, Redis, Cloud Map, ALB Routing)" %}}" style="text-decoration: none; font-weight: bold;">
⬅ BƯỚC 2: Infrastructure & ALB Setup
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.4-Task Definitions Creation (Configure settings, FIX environment variables)" %}}" style="text-decoration: none; font-weight: bold;">
BƯỚC 4: Task Definitions Creation ➡
</a>
</div>