+++
title = "Infrastructure & ALB Setup (RDS, Redis, Cloud Map, ALB Routing)"
weight = 2
chapter = false
pre = " <b> 5.3.2.2. </b> "
alwaysopen = true
+++


This phase provisions the data layer components, service discovery mechanism, and load balancing infrastructure.

### SSL Certificate Provisioning

**Step 1: Request ACM Certificate**

1.  Switch region to **Singapore (ap-southeast-1)**
2.  Navigate to **Certificate Manager (ACM)**
3.  Select **Request a certificate** → **Request a public certificate**
4.  Configure certificate request:
    -   **Domain name**: `api.sgutodolist.com`
    -   **Validation method**: DNS validation
5.  Click **Request**
6.  Access the certificate details → Under **Domains** section → Click **Create records in Route 53**
7.  Wait for status to change to **Issued** (typically 5-10 minutes)

* * * * *

### Data Layer Infrastructure

#### RDS MySQL Database

**Purpose**: Primary relational database for all microservices.

**Configuration Steps:**

1.  Navigate to **RDS Console** → **Create database**
2.  Select **Standard create** → **MySQL** engine
3.  Configure database instance:

| Parameter | Value | Rationale |
| --- | --- | --- |
| Template | Free tier | Cost optimization |
| DB instance identifier | `sgu-todolist-db` | Naming convention |
| Master username | `root` | Standard admin account |
| Master password | `[secure-password]` | Record securely |
| DB instance class | `db.t3.micro` | Free tier eligible |
| Allocated storage | 20 GiB | Free tier limit |

1.  Network configuration:
    -   **Compute resource**: Don't connect to an EC2 compute resource
    -   **VPC**: `SGU-Microservices-VPC`
    -   **Public access**: No
    -   **VPC security group**: Select `private-db-sg` (remove default)
    -   **Availability Zone**: `ap-southeast-1a`
2.  Click **Create database**
3.  After creation, record the **Endpoint** address (format: `sgu-todolist-db.[random].ap-southeast-1.rds.amazonaws.com`)

* * * * *

#### ElastiCache Redis

**Purpose**: Caching layer for session management, rate limiting, and temporary data storage.

**Configuration Steps:**

**Phase 1 - Basic Settings:**

1.  Navigate to **ElastiCache Console** → **Redis OSS caches** → **Create Redis OSS cache**
2.  Cluster configuration:

| Parameter | Value | Rationale |
| --- | --- | --- |
| Engine | Redis OSS | Open source compatible |
| Deployment option | Node-based cluster | Single node deployment |
| Creation method | Cluster cache | Standard creation |
| Cluster mode | Disabled | Simplified architecture |
| Cluster name | `sgu-redis` | Naming convention |
| Multi-AZ | Disabled | Cost optimization |
| Auto-failover | Disabled | Not required for development |
| Node type | `cache.t3.micro` | Free tier eligible |
| Number of replicas | 0 | Single node configuration |

1.  Network configuration:
    -   **Subnet groups**: Create a new subnet group
        -   **Name**: `sgu-redis-subnet-group`
        -   **VPC**: `SGU-Microservices-VPC`
        -   **Selected subnets**: Both Private Subnets
    -   **Availability Zone placement**: Specify Availability Zones
        -   **Primary**: `ap-southeast-1a`

**Phase 2 - Advanced Settings:**

1.  Security configuration:
    -   **Security groups**: Select `private-db-sg` (remove default)
2.  Backup configuration:
    -   **Enable automatic backups**: Disabled
3.  Logs: Disable all log types
4.  Click **Create**
5.  Record the **Primary Endpoint** (format: `sgu-redis.[random].cache.amazonaws.com`)

**Important:** Do not include `:6379` when recording the endpoint for environment variables.

* * * * *

#### Service Discovery Setup

**Purpose**: Internal DNS resolution for microservice-to-microservice communication.

**Step 1: Create Cloud Map Namespace**

1.  Navigate to **AWS Cloud Map** → **Create namespace**
2.  Configure namespace:
    -   **Name**: `sgu.local`
    -   **Instance discovery**: API calls and DNS queries in VPCs
    -   **VPC**: `SGU-Microservices-VPC`
3.  Click **Create**

**Result**: Internal DNS domain `sgu.local` is now available for service registration.

* * * * *

#### Apache Kafka Message Broker

**Purpose**: Event streaming and asynchronous communication between microservices.

**Step 1: Create Kafka Task Definition**

1.  Navigate to **Amazon ECS** → **Task definitions** → **Create new task definition**
2.  Infrastructure configuration:

| Parameter | Value |
| --- | --- |
| Task definition family | `kafka-server-td` |
| Launch type | AWS Fargate |
| Operating system | Linux/x86_64 |
| CPU | 0.5 vCPU |
| Memory | 1 GB |
| Task execution role | `ecsTaskExecutionRole` |

1.  Container configuration:
    -   **Name**: `kafka-server`
    -   **Image URI**: `bitnami/kafka:latest`
    -   **Port mappings**:
        -   Container port: `9092`
        -   Protocol: TCP
2.  Environment variables:

| Key | Value | Purpose |
| --- | --- | --- |
| `KAFKA_CFG_NODE_ID` | `0` | Node identifier |
| `KAFKA_CFG_PROCESS_ROLES` | `controller,broker` | Combined mode (KRaft) |
| `KAFKA_CFG_LISTENERS` | `PLAINTEXT://:9092,CONTROLLER://:9093` | Internal listeners |
| `KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP` | `CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT` | Protocol mapping |
| `KAFKA_CFG_CONTROLLER_QUORUM_VOTERS` | `0@127.0.0.1:9093` | Self-reference |
| `KAFKA_CFG_ADVERTISED_LISTENERS` | `PLAINTEXT://kafka.sgu.local:9092` | Service discovery address |
| `KAFKA_CFG_LOG_DIRS` | `/tmp/kafka-logs` | Temporary log storage |
| `ALLOW_PLAINTEXT_LISTENER` | `yes` | Non-SSL communication |

1.  Enable **Use log collection** for CloudWatch Logs
2.  Click **Create**

**Step 2: Deploy Kafka Service**

1.  Navigate to ECS Cluster → **Services** → **Create**
2.  Service configuration:
    -   **Task definition**: `kafka-server-td` (latest revision)
    -   **Service name**: `kafka-server`
    -   **Desired tasks**: 1
3.  Network configuration:
    -   **VPC**: `SGU-Microservices-VPC`
    -   **Subnets**: Both Public Subnets
    -   **Security group**: `private-db-sg`
    -   **Public IP**: Enabled
4.  Load balancing: Do not configure
5.  Service Discovery:
    -   Enable **Use service discovery**
    -   **Namespace**: `sgu.local`
    -   **Service discovery name**: `kafka`
    -   **Result DNS**: `kafka.sgu.local:9092`
6.  Click **Create**

* * * * *

#### Database Initialization

**Purpose**: Create application database schema securely through a jump server.

**Step 1: Update Security Group**

1.  Navigate to `private-db-sg` → **Inbound Rules** → **Edit**
2.  Add temporary rule:
    -   **Type**: MySQL/Aurora (port 3306)
    -   **Source**: `bastion-sg`
3.  Save rules

**Step 2: Create Bastion Host**

1.  Navigate to **EC2** → **Launch Instance**
2.  Configure instance:

| Parameter | Value |
| --- | --- |
| Name | `sgu-bastion` |
| AMI | Amazon Linux 2023 |
| Instance type | `t3.micro` |
| Key pair | Create new: `sgutodolist-key` (download .pem) |
| VPC | `SGU-Microservices-VPC` |
| Subnet | Public Subnet 1 |
| Auto-assign public IP | Enable |
| Security group | `bastion-sg` |

1.  Click **Launch instance**
2.  Record the **Public IPv4 address**

**Step 3: Connect and Initialize Database**

**Windows (PowerShell):**


```powershell

# Set correct permissions
icacls "sgutodolist-key.pem" /inheritance:r
icacls "sgutodolist-key.pem" /grant:r "$($env:USERNAME):(R)"

# Connect to Bastion
ssh -i "sgutodolist-key.pem" ec2-user@[BASTION-PUBLIC-IP]
```

**macOS/Linux:**


```bash

# Set correct permissions
chmod 400 sgutodolist-key.pem

# Connect to Bastion
ssh -i sgutodolist-key.pem ec2-user@[BASTION-PUBLIC-IP]
```

**On Bastion Host:**


```bash

# Install MySQL client
sudo dnf install mariadb105 -y

# Connect to RDS
mysql -h [RDS-ENDPOINT] -u root -p
# Enter password when prompted

# Create database
CREATE DATABASE aws_todolist_database
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;

# Verify
SHOW DATABASES;

# Exit
EXIT;
exit
```

**Step 4: Security Cleanup**

Remove temporary database access:

1.  Return to `private-db-sg` → **Inbound Rules**
2.  Remove the rule allowing `bastion-sg` access to port 3306

* * * * *

### Application Load Balancer Configuration

#### Target Groups Creation

Create 5 target groups for the microservices that will receive external traffic.

**Standard Configuration Process (Repeat 5 times):**

1.  Navigate to **EC2** → **Target Groups** → **Create target group**
2.  Basic configuration:
    -   **Target type**: IP addresses (required for Fargate)
    -   **Protocol**: HTTP
    -   **IP address type**: IPv4
    -   **VPC**: `SGU-Microservices-VPC`
    -   **Protocol version**: HTTP1
3.  Health check configuration:
    -   **Health check protocol**: HTTP
    -   **Health check path**: `/actuator/health`
4.  Click **Next** (do not register targets manually)
5.  Click **Create target group**

**Target Group Specifications:**

| # | Target Group Name | Port | Health Check Path | Purpose |
| --- | --- | --- | --- | --- |
| 1 | `api-gateway-tg` | 8080 | `/actuator/health` | API Gateway |
| 2 | `auth-tg` | 9999 | `/actuator/health` | Auth Service |
| 3 | `user-tg` | 8081 | `/actuator/health` | User Service |
| 4 | `task-tg` | 8082 | `/actuator/health` | Taskflow Service |
| 5 | `noti-tg` | 9998 | `/actuator/health` | Notification Service |

**Note:** AI Model Service does not need a target group as it is accessed only internally via API Gateway.

* * * * *

#### Load Balancer Creation

**Step 1: Create ALB**

1.  Navigate to **EC2** → **Load Balancers** → **Create load balancer**
2.  Select **Application Load Balancer**
3.  Basic configuration:

| Parameter | Value |
| --- | --- |
| Name | `sgu-alb` |
| Scheme | Internet-facing |
| IP address type | IPv4 |

1.  Network mapping:
    -   **VPC**: `SGU-Microservices-VPC`
    -   **Availability Zones**: Select both
        -   `ap-southeast-1a` → Public Subnet 1
        -   `ap-southeast-1b` → Public Subnet 2
2.  Security groups:
    -   Remove `default`
    -   Select `public-alb-sg`
3.  Listeners and routing:
    -   **Listener 1 (HTTP:80)**:
        -   Protocol: HTTP | Port: 80
        -   Default action: Forward to `api-gateway-tg`
    -   **Listener 2 (HTTPS:443)**: Click **Add listener**
        -   Protocol: HTTPS | Port: 443
        -   Default action: Forward to `api-gateway-tg`
        -   Certificate source: ACM
        -   Certificate: Select `api.sgutodolist.com`
4.  Click **Create load balancer**
5.  Wait for state to change from `Provisioning` to `Active`

* * * * *

#### Routing Rules Configuration

Configure path-based routing for the HTTPS listener.

**Step 1: Access Listener Rules**

1.  Select the ALB → **Listeners and rules** tab
2.  Select **HTTPS:443** listener → Click **Manage rules**

**Step 2: Add Routing Rules**

Create 4 rules with the following specifications:

| Priority | Rule Name | Path Pattern | Target Group |
| --- | --- | --- | --- |
| 1 | Auth Rule | `/api/auth/*` | `auth-tg` |
| 2 | User Rule | `/api/user/*` | `user-tg` |
| 3 | Task Rule | `/api/taskflow/*` | `task-tg` |
| 4 | Noti Rule | `/api/notification/*` | `noti-tg` |

**Rule Creation Process (Repeat 4 times):**

1.  Click **Add rule**
2.  **Step 1 - Define rule**:
    -   **Name**: Enter rule name (e.g., "Auth Rule")
    -   **Conditions**: Add condition → Path → Enter path pattern (e.g., `/api/auth/*`)
    -   **Actions**: Forward to target groups → Select corresponding target group
    -   Click **Next**
3.  **Step 2 - Set rule priority**:
    -   **Priority**: Enter priority number (1-4)
    -   Click **Add rule**

**Verification**: The default rule forwarding to `api-gateway-tg` should remain at the bottom with priority "Last".

* * * * *

### Infrastructure Validation

Before proceeding to code deployment, verify:

**Data Layer:**

-   [ ]  RDS instance status: Available
-   [ ]  RDS endpoint recorded
-   [ ]  Database `aws_todolist_database` created

**Caching Layer:**

-   [ ]  Redis cluster status: Available
-   [ ]  Redis endpoint recorded (without :6379)

**Service Discovery:**

-   [ ]  Cloud Map namespace `sgu.local` created
-   [ ]  Kafka service running with DNS `kafka.sgu.local`

**Load Balancer:**

-   [ ]  ALB status: Active
-   [ ]  5 target groups created
-   [ ]  HTTPS listener configured with ACM certificate
-   [ ]  4 path-based routing rules configured

* * * * *
