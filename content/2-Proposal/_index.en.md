+++ 
title = "Proposal"
weight = 2
chapter = true
pre = "<b>2. </b>" 
+++

Cost-Optimized SaaS Task Management Platform
============================================

Single-Region High Availability with ECS Fargate
------------------------------------------------

* * * * *

### **1\. Executive Summary**

The SaaS Task Management Platform is designed to deliver a Todoist-like collaborative experience with **High Availability (HA)** and **Cost Efficiency**, specifically optimized for **AWS Free Tier** constraints.

Initially, the project targeted a **Multi-Region deployment** to achieve the high levels of global performance, disaster recovery (DR), and uptime. However, due to the **cost limitations** and the constraints of the **AWS Free Tier** (particularly concerning cross-region data transfer and running multiple primary/replica database instances), the architecture was strategically consolidated into a **Single AWS Region (ap-southeast-1 - Singapore)**.

Built on **Spring Boot microservices** deployed as **ECS Fargate containers**, the platform now leverages Multi-AZ deployment within Singapore for resilience while eliminating server management overhead.

**Key Architecture Highlights:**

-   **Serverless Compute:** ECS Fargate eliminates EC2 instance management

-   **Regional High Availability:** Multi-AZ deployment for compute (ECS), database (RDS), and caching (ElastiCache)

-   **Low Latency in SEA:** Optimized performance for Southeast Asian users

-   **Global Content Delivery:** CloudFront CDN for worldwide static asset distribution

-   **Cost Predictability:** Architecture designed to maximize AWS Free Tier benefits

-   **Disaster Recovery:** S3 Cross-Region Replication to us-east-1 for data backup

The result is a production-ready, cost-effective platform that balances performance, availability, and operational simplicity---ideal for MVP development and regional deployment, while maintaining a clear roadmap for **future Multi-Region expansion**.
* * * * *

### **2\. Problem Statement & Solution**

#### **2.1. The Challenge: Balancing Performance, Availability, and Cost**

Traditional SaaS platforms face critical trade-offs when operating on limited budgets:

**Operational Complexity:**

-   EC2-based deployments require constant server management (patching, monitoring, capacity planning)

-   Manual scaling decisions lead to over-provisioning (wasted money) or under-provisioning (poor performance)

-   Complex deployment processes prone to human error and downtime

**Cost Challenges:**

-   Multi-region architectures introduce expensive cross-region data transfer

-   Running RDS Read Replicas in multiple regions quickly exhausts Free Tier limits

-   NAT Gateway charges ($35+/month) consume significant portions of small budgets

-   Always-on EC2 instances waste capacity during off-peak hours

**Availability vs. Budget:**

-   Single-region deployments risk total outage during regional failures

-   Multi-AZ configurations increase costs but are essential for production workloads

-   Balancing HA requirements with Free Tier constraints requires careful architecture

#### **2.2. The Single-Region ECS Solution**

This project adopts a **Single-Region (ap-southeast-1) High Availability** architecture using **ECS Fargate** for serverless compute, optimizing for cost while maintaining production-grade reliability.

**Core Architecture Decisions:**

**Compute Layer (Serverless):**

-   **ECS Fargate** for containerized Spring Boot microservices (no EC2 management)

-   **Application Load Balancer** for traffic distribution and health checks

-   **AWS Cloud Map** for service discovery between microservices

-   **Auto Scaling** based on CPU/memory metrics

**Data Layer (Regional + Multi-AZ):**

-   **RDS MySQL (db.t3.micro)** with Multi-AZ for automated failover

-   **ElastiCache Redis (cache.t3.micro)** for session management and caching

-   **S3** with Cross-Region Replication to us-east-1 for disaster recovery

**Network Layer (Global + Regional):**

-   **VPC** with public/private subnets across 2 Availability Zones

-   **CloudFront CDN** for global static asset delivery

-   **Route 53** for DNS management and domain hosting

-   **VPC Endpoints** to eliminate NAT Gateway costs ($35/month savings)

**Security & Observability:**

-   **Security Groups** for network-level access control

-   **IAM Roles** for service-to-service authentication (no hardcoded credentials)

-   **Origin Access Control (OAC)** to secure S3 buckets

-   **CloudWatch** for centralized logging, metrics, and alarms

**Key Advantages Over Traditional Architecture:**

| **Aspect** | **Traditional EC2** | **ECS Fargate (This Project)** |
| --- | --- | --- |
| Server Management | Manual patching, monitoring, SSH access | Fully managed, no server access needed |
| Scaling | Manual ASG configuration | Automatic based on metrics |
| Deployment | Complex, multi-step process | Rolling updates with zero downtime |
| Cost Model | Pay for running instances (24/7) | Pay only for container runtime |
| Free Tier | 750 EC2 hours/month | 20 GB-hours + 50 GB data transfer |
| Startup Time | 2-5 minutes (AMI boot) | 30-60 seconds (container start) |

#### **2.3. Benefits and ROI**

**Technical Benefits:**

-   **80% reduction** in operational overhead (no server management)

-   **99.5%+ uptime** with Multi-AZ deployment for RDS and ECS

-   **<100ms API response time** for cached requests

-   **2-5 minute** recovery time for AZ failures (automatic)

**Cost Benefits:**

-   **~$25-30/month** operational cost (vs $50-90 with EC2)

-   **VPC Endpoints** eliminate $35/month NAT Gateway charges

-   **Pay-per-use** pricing---only charged for actual container runtime

-   **Free Tier optimized** to stay within budget constraints

**Learning & Career Value:**

-   Hands-on experience with modern cloud-native architecture

-   Container orchestration expertise (Docker, ECS Fargate)

-   DevOps practices: IaC, CI/CD, monitoring, incident response

-   Portfolio project demonstrating AWS Solutions Architect competencies

* * * * *

### **3\. Solution Architecture**

<!-- {{< figurecaption src="/images/todolist-architecture-6.jpg" caption="Figure 1. Single-Region ECS Fargate Architecture" >}} -->
<img src="/images/todolist-architecture-6.jpg" alt="Figure 1. Single-Region ECS Fargate Architecture" style="max-width:100%;height:auto;">
<figcaption>Figure 1. Single-Region ECS Fargate Architecture</figcaption>


#### **3.1. Architecture Overview**

**Primary Deployment Region: ap-southeast-1 (Singapore)**

*Compute & Orchestration:*

-   **ECS Fargate Cluster** running 5 containerized Spring Boot microservices

-   **Application Load Balancer** distributing traffic across ECS tasks (Multi-AZ)

-   **AWS Cloud Map** for internal service discovery (private DNS namespace)

-   **Auto Scaling policies** targeting 70% CPU, 80% memory utilization

*Data & Caching:*

-   **RDS MySQL (db.t3.micro)** with Multi-AZ deployment for automatic failover

-   **ElastiCache Redis (cache.t3.micro)** for session storage and hot data caching

-   **S3 bucket** for user files, attachments, and static assets

*Network Infrastructure:*

-   **VPC (10.0.0.0/16)** with public and private subnets across 2 Availability Zones

-   **Internet Gateway** for public subnet (ALB) internet connectivity

-   **VPC Endpoints** for S3, ECR, CloudWatch (eliminates NAT Gateway cost)

-   **Security Groups** controlling all traffic flows between components

**Global Services:**

*Content Delivery:*

-   **CloudFront CDN** with global edge locations for static asset distribution

-   **Origin:** Primary S3 bucket in Singapore, failover to us-east-1

-   **Origin Access Control (OAC)** ensuring S3 is only accessible via CloudFront

*Disaster Recovery:*

-   **S3 Cross-Region Replication (CRR)** to us-east-1 (N. Virginia)

-   **RDS Automated Snapshots** (7-day retention, daily backups)

*DNS & Security:*

-   **Route 53** for domain hosting and DNS management

-   **ACM (AWS Certificate Manager)** for free SSL/TLS certificates

-   **CloudWatch** for centralized logs, metrics, and alarms

#### **3.2. Microservices Architecture**

The platform implements **5 core Spring Boot microservices** deployed as ECS Fargate tasks:

| **Service** | **Port** | **Responsibilities** | **Container Resources** |
| --- | --- | --- | --- |
| **API Gateway** | **8080** | API Traffic Routing, Request Aggregation/Validation | 0.5 vCPU, 1GB RAM |
| **User Service** | **8081** | User profiles, settings, avatar management | 0.5 vCPU, 1GB RAM |
| **Taskflow Service** | **8082** | Board/workspace creation, Task CRUD, assignments, comments, attachments | 0.5 vCPU, 1GB RAM |
| **Notification Service** | **9998** | Real-time notifications, email (SES), WebSockets | 0.5 vCPU, 1GB RAM |
| **Auth Service** | **9999** | User authentication, JWT tokens, OAuth2 (Google) | 0.5 vCPU, 1GB RAM |

**Service Communication Patterns:**

*External Traffic (Users):*

```
User → Route 53 → CloudFront (static) OR ALB (API)
    → ALB Path-Based Routing:
      /api/* → API Gateway Target Group (Port 8080)

API Gateway Internal Routing (Port-based, via Cloud Map):
    /auth/*   → Auth Service (Port 9999)
    /users/*  → User Service (Port 8081)
    /taskflow/*  → Taskflow Service (Port 8082)
    /notifications/* → Notification Service (Port 9998)

```

*Internal Service-to-Service:*

```
Service A → AWS Cloud Map (service-b.local:PORT)
          → Direct container-to-container communication
          → Response cached in Redis (5-minute TTL)

```

**Container Configuration:**

-   **Base Image:** `eclipse-temurin:21-jre-alpine` (lightweight JRE)

-   **Health Check:** Spring Actuator `/actuator/health/liveness` endpoint (used by ALB)

-   **Logging:** CloudWatch Logs with 7-day retention

-   **Environment Variables:** Database credentials, Redis endpoint, S3 bucket name

-   **IAM Task Role:** Permissions for S3, SES, CloudWatch access

#### **3.3. AWS Services Used**

| **Category** | **Services** | **Purpose** | **Cost Optimization** |
| --- | --- | --- | --- |
| **Compute** | ECS Fargate, ALB | Serverless container orchestration, load balancing | Free Tier: 750 ALB hours, 20 GB Fargate hours |
| **Database** | RDS MySQL (db.t3.micro) | Primary database with Multi-AZ HA | Fixed cost: ~$15/month (exceeds Free Tier) |
| **Caching** | ElastiCache Redis (cache.t3.micro) | Session store, API response caching | Free Tier: 750 node hours |
| **Storage** | S3 Standard + CRR | Object storage with DR replication | Free Tier: 5GB storage, 20k GET, 2k PUT |
| **CDN** | CloudFront | Global content delivery | Free Tier: 1TB data transfer, 10M requests |
| **Container Registry** | ECR | Docker image storage | Free Tier: 500MB storage |
| **Service Discovery** | AWS Cloud Map | Microservices service registry | $0.50/namespace + $0.10/instance/month |
| **Networking** | VPC, VPC Endpoints | Network isolation, private AWS service access | VPC free, Endpoints ~$7/month (replaces $35 NAT) |
| **DNS** | Route 53 | Domain hosting | $0.50/hosted zone |
| **Security** | ACM, IAM, Security Groups | SSL certificates, access control | Free |
| **Observability** | CloudWatch Logs, Metrics, Alarms | Monitoring, logging, alerting | Free Tier: 5GB logs, 10 custom metrics |

* * * * *

### **4\. Service Roles Overview**

| **AWS Service** | **Role in Architecture** | **Configuration Details** |
| --- | --- | --- |
| **Route 53** | DNS hosting for custom domain, SSL certificate validation | Hosted zone: `sgutodolist.com`, Health checks for ALB |
| **CloudFront** | Global CDN serving static assets from edge locations | Origin: S3 Singapore (primary), S3 Virginia (failover) |
| **ACM** | Free SSL/TLS certificates for HTTPS | Wildcard cert: `*.sgutodolist.com`, auto-renewal enabled |
| **VPC** | Isolated network for all resources | CIDR: 10.0.0.0/16, 2 public + 2 private subnets across 2 AZs |
| **Internet Gateway** | Enables ALB to receive traffic from internet | Attached to public subnets only |
| **VPC Endpoints** | Private connections to AWS services (no NAT Gateway) | S3 Gateway Endpoint, Interface Endpoints for ECR/CloudWatch |
| **Application Load Balancer** | Layer 7 routing, SSL termination, health checks | Multi-AZ, **single routing rule to API Gateway Target Group** |
| **ECS Fargate** | Serverless container runtime (no EC2 management) | Runs Spring Boot containers with varying ports |
| **ECS Cluster** | Logical grouping of ECS services and tasks | Single cluster: `sgutodolist-cluster` |
| **ECS Service** | Maintains desired task count, integrates with ALB | Desired count: 2 tasks/service, rolling deployment strategy |
| **ECS Task Definition** | Blueprint for containers (image, resources, environment) | 5 task definitions (one per microservice) |
| **AWS Cloud Map** | Service discovery via private DNS | Namespace: `local`, services: `api-gateway.local`, `auth-service.local`, etc. |
| **RDS MySQL** | Primary relational database with Multi-AZ | db.t3.micro, 20GB storage, automated backups enabled |
| **ElastiCache Redis** | In-memory cache for sessions and API responses | cache.t3.micro, cluster mode disabled |
| **S3 (Primary)** | User files, attachments, static frontend assets | Bucket: `sgutodolist-assets-sg` (ap-southeast-1) |
| **S3 (Secondary)** | Disaster recovery replica | Bucket: `sgutodolist-assets-us` (us-east-1) |
| **S3 CRR** | Automated async replication Singapore → Virginia | Replication rule for all objects, versioning enabled |
| **Origin Access Control** | Restricts S3 access to CloudFront only | Blocks direct public access to S3 buckets |
| **Security Groups** | Virtual firewall for ALB, ECS tasks, RDS, Redis | Least-privilege rules, source-based restrictions |
| **IAM Task Execution Role** | Allows ECS to pull images from ECR, write logs | Permissions: ECR pull, CloudWatch Logs write |
| **IAM Task Role** | Allows containers to access S3, SES, etc. | Permissions: S3 read/write, SES send, CloudWatch metrics |
| **CloudWatch Logs** | Centralized application logs from ECS containers | 7-day retention, 5 log groups (one per service) |
| **CloudWatch Metrics** | Performance metrics (CPU, memory, request count) | Custom metrics for business KPIs |
| **CloudWatch Alarms** | Alerting for high CPU, failed tasks, budget thresholds | SNS integration for email notifications |

* * * * *

### **5\. Service Flow**

#### **5.1. User Request Flow**

**Static Assets (Frontend - HTML, CSS, JS, Images):**

1.  User accesses `https://sgutodolist.com`

2.  Route 53 resolves DNS to CloudFront distribution

3.  CloudFront checks nearest edge location cache:

    -   **Cache HIT:** Returns asset immediately (latency <20ms)

    -   **Cache MISS:** Fetches from S3 origin (Singapore), caches for 24 hours

4.  Browser loads frontend application

5.  Static assets served with low latency globally via edge locations

**API Requests (Backend Microservices):**

1.  Frontend makes API call: `https://sgutodolist.com/api/task`

2.  Route 53 resolves to Application Load Balancer in Singapore

3.  ALB performs SSL termination and routes all `/api/*` traffic to the **API Gateway Target Group (Port 8080)**.

4.  API Gateway (running on ECS) receives the request and **internally routes** via AWS Cloud Map based on the path:

    -   Requests to `/api/auth/**` are routed to **Auth Service (Port 9999)**.

    -   Requests to `/api/user/**` are routed to **User Service (Port 8081)**.

    -   Requests to `/api/taskflow/**` are routed to **Taskflow Service (Port 8082)**.

    -   Requests to `/api/notifications/**` are routed to **Notification Service (Port 9998)**.

5.  The target microservice processes the request.

**Data Access Pattern (90% Reads, 10% Writes):**

*Read Operations:*

1.  Spring Boot service receives request

2.  Check ElastiCache Redis for cached data:

    -   **Cache HIT:** Return immediately (latency <5ms)

    -   **Cache MISS:** Query RDS MySQL (Multi-AZ), cache result with TTL

3.  Response returned through ALB to user

4.  Total latency: 5-10ms (cached) or 20-50ms (database query)

*Write Operations:*

1.  Spring Boot service validates request

2.  Write to RDS MySQL primary database

3.  RDS synchronously replicates to standby instance (same region, different AZ)

4.  Invalidate/update related cache entries in Redis

5.  Async: Trigger notifications, update search index

6.  Success response to user

7.  Total latency: 50-100ms

**Service-to-Service Communication:**

Example: API Gateway needs to validate a JWT token via Auth Service

1.  API Gateway queries AWS Cloud Map: `http://auth-service.local:9999/validate`

2.  Cloud Map resolves to healthy Auth Service task private IP

3.  Direct container-to-container communication within VPC (no ALB overhead)

4.  Response cached in Redis for 5 minutes to reduce repeated calls

5.  Latency: 10-20ms for first call, <5ms for subsequent cached calls

#### **5.2. Developer Deployment Flow**

**CI/CD Pipeline (Manual Process, Automatable with CodePipeline):**

**Phase 1: Local Development**

1.  Developer updates Spring Boot microservice code

2.  Run unit tests: `./mvnw test`

3.  Run integration tests with Docker Compose (local MySQL + Redis)

4.  Commit changes to Git repository

**Phase 2: Container Build**

1.  Build Spring Boot JAR: `./mvnw clean package`

2.  Build Docker image:

    ```
    FROM eclipse-temurin:21-jre-alpineWORKDIR /appCOPY target/task-service.jar app.jarEXPOSE 8084HEALTHCHECK CMD curl -f http://localhost:8084/actuator/health || exit 1ENTRYPOINT ["java", "-jar", "app.jar"]

    ```

3.  Tag image: `task-service:v1.2.3`

**Phase 3: Push to ECR**

```
# Authenticate to ECR
aws ecr get-login-password --region ap-southeast-1 |\
  docker login --username AWS --password-stdin {account-id}.dkr.ecr.ap-southeast-1.amazonaws.com

# Tag and push
docker tag task-service:v1.2.3\
  {account-id}.dkr.ecr.ap-southeast-1.amazonaws.com/task-service:v1.2.3
docker push {account-id}.dkr.ecr.ap-southeast-1.amazonaws.com/task-service:v1.2.3

```

**Phase 4: Update ECS Task Definition**

1.  Navigate to ECS console → Task Definitions → `task-service`

2.  Create new revision:

    -   Update container image to `:v1.2.3`

    -   Review environment variables (DB_HOST, REDIS_HOST, etc.)

    -   Verify resource allocation (0.5 vCPU, 1GB RAM)

3.  Register new task definition revision

**Phase 5: Deploy to ECS Service (Rolling Update)**

1.  Navigate to ECS Service → `task-service`

2.  Update service to use new task definition revision

3.  ECS performs automatic rolling update:

    -   Launch 2 new tasks with updated image

    -   Wait for tasks to pass ALB health checks (3 consecutive passes)

    -   ALB starts routing 50% traffic to new tasks

    -   Drain connections from old tasks (30-second timeout)

    -   Stop old tasks once fully drained

4.  Total deployment time: 3-5 minutes (zero downtime)

**Phase 6: Verification & Monitoring**

1.  Check ECS Service events: `aws ecs describe-services`

2.  Monitor CloudWatch Logs for errors: `/ecs/task-service`

3.  Test API endpoints via ALB: `curl https://sgutodolist.com/api/task/health`

4.  Check CloudWatch Metrics: CPU, memory, request count, error rate

5.  Monitor ALB Target Health in console

**Rollback Procedure (If Issues Detected):**

1.  Identify last known good task definition revision (e.g., `v1.2.2`)

2.  Update ECS Service to use previous revision

3.  ECS performs automatic rollback deployment (3-5 minutes)

4.  Verify health checks pass and logs show no errors

5.  Investigate issue in lower environments before next deployment

**Future Automation (Phase 2 Roadmap):**

-   **GitHub Actions:** Trigger builds on push to `main` branch

-   **AWS CodePipeline:** Source (GitHub) → Build (CodeBuild) → Deploy (ECS)

-   **Blue/Green Deployments:** Use CodeDeploy for safer production rollouts

-   **Automated Testing:** Integration tests run before deployment

#### **5.3. Data Replication & Storage Strategy**

**Data Storage & Backup Matrix:**

| **Data Type** | **Primary Storage** | **Replication Method** | **Backup** | **RPO** | **RTO** |
| --- | --- | --- | --- | --- | --- |
| **Transactional Data** (users, boards, tasks) | RDS MySQL Multi-AZ | Synchronous (within AZs) | Automated snapshots (daily, 7-day retention) | <1 minute | 10-20 minutes (AZ failure), 2-4 hours (region failure) |
| **Session Tokens** | ElastiCache Redis | None (ephemeral) | None (regenerate on failure) | N/A | Immediate (users re-authenticate) |
| **User Files** (attachments, avatars) | S3 Singapore | CRR to S3 Virginia (async) | Versioning (30-day retention) | 5-15 minutes | Immediate (CloudFront failover) |
| **Static Assets** (frontend code) | S3 Singapore | CRR to S3 Virginia (async) | Versioning + git repository | 5-15 minutes | Immediate (CloudFront failover) |
| **Application Logs** | CloudWatch Logs | Regional replication (managed by AWS) | 7-day retention | N/A | N/A |
| **Docker Images** | ECR Singapore | Manual push to other regions if needed | Image versioning | N/A | Minutes (pull from ECR) |

**S3 Cross-Region Replication Configuration:**

-   **Source Bucket:** `sgutodolist-assets-sg` (ap-southeast-1)

-   **Destination Bucket:** `sgutodolist-assets-us` (us-east-1)

-   **Replication Rules:**

    -   Replicate all objects (prefix: `/`)

    -   Replicate metadata, tags, and object ACLs

    -   **Do NOT** replicate delete markers (prevent accidental data loss)

    -   Priority: `1` (highest)

-   **Versioning:** Enabled on both source and destination

-   **Expected Latency:** 5-15 minutes for most objects (<1MB)

-   **Use Case:** Disaster recovery + CloudFront origin failover

#### **5.4. High Availability & Disaster Recovery**

**Failure Scenarios & Automated Response:**

**Scenario 1: Single ECS Task Failure**

-   **Detection:** ALB health check fails (3 consecutive failures at 10-second intervals)

-   **Automatic Response:**

    -   ALB stops routing new requests to unhealthy task

    -   Existing connections drained (30-second timeout)

    -   ECS Service Controller launches replacement task automatically

-   **Impact:** Zero user-facing downtime (other tasks handle load)

-   **Recovery Time:** 2-3 minutes (container start + health check pass)

-   **User Experience:** No interruption

**Scenario 2: Complete Service Failure (All Tasks Unhealthy)**

-   **Detection:** All 2 tasks for a service fail health checks, ALB returns 503 errors

-   **Automatic Response:** ECS attempts to launch replacement tasks

-   **Manual Intervention Required:**

    -   Check CloudWatch Logs for root cause (database connection timeout, memory leak, bad deployment)

    -   If bad deployment: Rollback to previous task definition revision

    -   If infrastructure issue: Check RDS/Redis connectivity, Security Groups

-   **Impact:** Service unavailable until new tasks healthy or issue resolved

-   **Recovery Time:** 5-15 minutes (depends on issue complexity)

-   **Mitigation:** Implement circuit breakers, graceful degradation

**Scenario 3: Availability Zone Failure (e.g., AZ-1a Goes Down)**

-   **Detection:** All tasks in AZ-1a become unreachable, ALB marks them unhealthy

-   **Automatic Response:**

    -   ALB immediately routes all traffic to tasks in healthy AZ (AZ-1b)

    -   ECS Service launches replacement tasks in healthy AZs to restore desired count

    -   RDS Multi-AZ: If primary DB in failed AZ, RDS automatically fails over to standby (60-120 seconds)

-   **Impact:**

    -   30-50% capacity reduction for 5-10 minutes (temporary latency increase)

    -   Possible 60-120 second database unavailability during RDS failover

-   **Recovery Time:** 5-10 minutes for full capacity restoration

-   **User Experience:** Slight performance degradation, no data loss

**Scenario 4: RDS Primary Database Failure**

-   **Detection:**

    -   RDS health checks fail

    -   Application logs show connection timeouts to database

    -   CloudWatch alarm triggers: `DatabaseConnections` metric drops to 0

-   **Automatic Response:**

    -   RDS Multi-AZ automatically promotes standby instance to primary

    -   DNS endpoint (`sgutodolist-db.xxxxx.ap-southeast-1.rds.amazonaws.com`) updated to point to new primary

    -   Application connection pools automatically reconnect (Spring Boot retry logic)

-   **Impact:** 60-120 seconds of database write unavailability

-   **Recovery Time:** 1-2 minutes (fully automated)

-   **Data Loss:** Zero (synchronous replication to standby)

**Scenario 5: Complete Regional Failure (Singapore Outage)**

-   **Detection:**

    -   Route 53 health checks fail for Singapore ALB

    -   CloudWatch alarms trigger: All ECS tasks unreachable

    -   Manual verification: AWS Service Health Dashboard confirms regional issue

-   **Manual DR Process:**

    *Option A: Read-Only Mode (15-30 minutes)*

    1.  Update CloudFront origin to use S3 Virginia bucket (static assets still work)

    2.  Display maintenance page to users: "Service temporarily unavailable"

    3.  Wait for AWS to restore Singapore region

    *Option B: Full Recovery to New Region (2-4 hours)*

    1.  Restore latest RDS snapshot to new region (us-east-1):

        ```
        aws rds restore-db-instance-from-db-snapshot \  --db-instance-identifier sgutodolist-dr \  --db-snapshot-identifier rds:sgutodolist-db-2024-12-07-00-00 \  --db-instance-class db.t3.micro \  --availability-zone us-east-1a

        ```

    2.  Create new ECS Fargate cluster in us-east-1

    3.  Deploy all 5 microservices using existing task definitions (update DB endpoint)

    4.  Create new ALB in us-east-1, configure target groups

    5.  Update Route 53 to point to new ALB

    6.  Update CloudFront origin to use new ALB for API calls

-   **Impact:** Full service outage during recovery

-   **Recovery Time:** 2-4 hours (manual process)

-   **Data Loss:** Last 5-60 minutes (RDS automated backups every 5 minutes, snapshots hourly)

-   **Cost:** Additional resources in us-east-1 (~$30/month if kept running)

**Disaster Recovery Metrics:**

-   **RPO (Recovery Point Objective):** <1 hour (RDS automated backups)

-   **RTO (Recovery Time Objective):**

    -   AZ failure: 5-10 minutes (automatic)

    -   Regional failure: 2-4 hours (manual)

-   **Availability Target:** 99.5% (43 minutes downtime/month allowance)

* * * * *

### **6\. Budget Estimation**

#### **6.1. Monthly Cost Breakdown (Free Tier Optimized)**

| **AWS Service** | **Specification** | **Free Tier Allocation** | **Expected Usage** | **Cost/Month** |
| --- | --- | --- | --- | --- |
| **ECS Fargate** | 5 services x 2 tasks x 0.5 vCPU, 1GB RAM | 20 GB-hours/month | 10 tasks x 24h x 30d = 7,200 GB-hours | $0 (Month 1), ~$36 after Free Tier |
| **RDS MySQL** | db.t3.micro, Multi-AZ, 20GB storage | 750 hours Single-AZ only | 744 hours Multi-AZ | **$15.00** (fixed cost) |
| **ElastiCache Redis** | cache.t3.micro, single node | 750 node hours | 744 hours | $0 (Free Tier) |
| **Application Load Balancer** | Standard ALB, minimal LCUs | 750 hours + 15 LCUs | 744 hours, 10 LCUs | $0 (Free Tier) |
| **S3 Storage** | Standard class + CRR | 5GB storage, 20k GET, 2k PUT | 10GB storage, 30k GET, 5k PUT, CRR | **$2.00** |
| **CloudFront** | Global edge locations | 1TB data transfer, 10M requests | 50GB data transfer, 2M requests | $0 (Free Tier) |
| **Route 53** | Hosted zone + queries | First 25 zones discounted | 1 hosted zone, 1M queries | **$0.50** |
| **VPC Endpoints** | S3 Gateway + ECR/CloudWatch Interface | None | 3 endpoints x 24h x 30d | **$7.00** |
| **ECR** | Docker image storage | 500MB storage/month | 2GB storage (5 images x 400MB) | **$0.20** |
| **AWS Cloud Map** | Service discovery namespace | None | 1 namespace + 5 service instances | **$1.00** |
| **CloudWatch** | Logs, metrics, alarms | 5GB logs, 10 custom metrics | 3GB logs (7-day retention), 15 metrics, 5 alarms | **$3.00** |
| **Data Transfer** | Inter-AZ, internet egress | 100GB out to internet | 30GB out (API responses, ALB traffic) | **$3.00** |
| **VPC, Security Groups, IAM, ACM** | Networking and security | Free | N/A | **$0.00** |
|   |   |   | **Month 1 Total:** | **~$31.70** |
|   |   |   | **Month 2+ Total:** | **~$67.70** |

#### **6.2. Cost Optimization Strategies**

**Immediate Savings (Implemented):**

1.  **✅ VPC Endpoints Instead of NAT Gateway (-$28/month)**

    -   **Before:** NAT Gateway ($32/month) + data processing ($3/month) = $35/month

    -   **After:** VPC Endpoints ($7/month for 3 endpoints)

    -   **Savings:** $28/month ($336/year)

    -   **Trade-off:** None---endpoints are more secure and faster

2.  **✅ Single-Region Deployment (-$50+/month)**

    -   **Before:** Multi-region (Singapore + Sydney) with cross-region data transfer

    -   **After:** Single region with S3 CRR for DR only

    -   **Savings:** No second RDS instance ($15), no second ElastiCache ($12), no second ALB ($16), no cross-region data transfer ($10-20)

    -   **Trade-off:** Users outside SEA experience higher API latency (acceptable for MVP)

3.  **✅ ECS Fargate with Minimal Resources (-$10-15/month vs larger EC2)**

    -   **Configuration:** 0.5 vCPU, 1GB RAM per task (Fargate minimum)

    -   **Savings:** Efficient resource allocation, pay only for what you use

    -   **Trade-off:** Monitor performance, scale up if needed

**Additional Optimizations (Consider if Budget Exceeded):**

1.  **Reduce ECS Task Count During Off-Peak Hours**

    -   Current: 2 tasks per service (10 total) running 24/7

    -   Optimization: Scale down to 1 task per service during 12am-8am SGT

    -   **Potential Savings:** ~$6/month

    -   **Risk:** Reduced redundancy during off-hours

2.  **Optimize CloudWatch Log Retention**

    -   Current: 7-day retention (3GB logs)

    -   Optimization: Reduce to 3-day retention

    -   **Potential Savings:** ~$1-2/month

    -   **Trade-off:** Shorter log history for debugging

3.  **Use S3 Intelligent-Tiering**

    -   Automatically moves infrequently accessed objects to cheaper storage tiers

    -   **Potential Savings:** $0.50-1/month

    -   **Trade-off:** Minimal retrieval delay for cold objects

4.  **Disable RDS Multi-AZ Temporarily (NOT RECOMMENDED)**

    -   **Savings:** ~$7.50/month (50% reduction)

    -   **CRITICAL RISK:** No automatic failover, accept downtime during DB failures

    -   **Use Case:** Only for development/testing environments

**Revised Budget Scenarios:**

| **Scenario** | **Monthly Cost** | **Annual Cost** | **Use Case** |
| --- | --- | --- | --- |
| **Current (Free Tier - Month 1)** | $31.70 | - | Initial launch with Free Tier benefits |
| **Standard (Post Free Tier)** | $67.70 | $812/year | Production after Free Tier expires |
| **Optimized (Off-peak scaling)** | $61.70 | $740/year | Budget-conscious production |
| **Development (No Multi-AZ)** | $60.20 | $722/year | Testing environment only |

#### **6.3. Free Tier Monitoring & Budget Alerts**

**Critical Free Tier Limits to Track:**

| **Service** | **Free Tier Limit** | **Monthly Allowance** | **Alert Threshold** | **Monitoring Method** |
| --- | --- | --- | --- | --- |
| **ECS Fargate** | 20 GB-hours | First month only | 16 GB-hours (80%) | CloudWatch custom metric + AWS Budgets |
| **RDS** | 750 hours | Single-AZ only (Multi-AZ NOT covered) | N/A (always paid) | N/A |
| **ElastiCache** | 750 node-hours | Monthly (cache.t2.micro or t3.micro) | 700 hours (93%) | CloudWatch billing metric |
| **ALB** | 750 hours + 15 LCUs | Monthly | 700 hours (93%) | AWS Cost Explorer |
| **S3** | 5GB storage, 20k GET, 2k PUT | Monthly | 4GB, 18k GET, 1.8k PUT | S3 Storage Lens |
| **CloudFront** | 1TB data transfer, 10M requests | Monthly | 900GB, 9M requests | CloudFront usage reports |
| **Data Transfer** | 100GB out to internet | Monthly | 90GB (90%) | CloudWatch billing alarm |

**AWS Budgets Configuration:**

*Budget 1: Overall Monthly Budget*

-   **Name:** "Production Infrastructure Budget"

-   **Amount:** $70/month

-   **Alert Thresholds:**

    -   50% ($35) - Email to team

    -   80% ($56) - Email to admin + Slack notification

    -   100% ($70) - Email + SMS to admin

    -   110% ($77) - Email + trigger cost reduction script

*Budget 2: ECS Fargate Specific*

-   **Name:** "ECS Fargate Monitor"

-   **Amount:** $40/month

-   **Filtered by:** Service = ECS

-   **Alert Thresholds:** 80%, 100%

*Budget 3: Data Transfer Watch*

-   **Name:** "Data Transfer Overage Prevention"

-   **Amount:** $10/month

-   **Filtered by:** Usage Type Group = Data Transfer

-   **Alert Thresholds:** 50%, 80%, 100%

**Daily Monitoring Routine (5 minutes):**

1.  Check AWS Cost Explorer → Daily spend trend

2.  Review ECS Service metrics → Task count hasn't scaled unexpectedly

3.  Verify S3 data transfer → No unusual spikes from CRR

4.  Check CloudWatch alarms → No billing alerts triggered

5.  Review top 5 cost drivers → Identify any anomalies

**Weekly Cost Review (15 minutes):**

1.  Generate cost report by service (last 7 days)

2.  Compare to previous week → Identify trends

3.  Review CloudFront data transfer → Validate CDN efficiency

4.  Check RDS Performance Insights → Optimize slow queries

5.  Update cost forecast for end of month

* * * * *

### **7\. Risk Assessment**

#### **7.1. Risk Matrix**

| **Risk Category** | **Specific Risk** | **Impact** | **Probability** | **Priority** | **Mitigation Strategy** |
| --- | --- | --- | --- | --- | --- |
| **Cost** | Free Tier exhaustion before month end | High | High | **CRITICAL** | Daily monitoring, budget alarms at 50%/80%/100%, auto-scaling limits |
| **Cost** | ECS Fargate over-scaling during traffic spike | High | Medium | **HIGH** | Set maximum task count to 3 per service, configure target tracking conservatively |
| **Cost** | Unexpected data transfer charges | Medium | Medium | **MEDIUM** | VPC Endpoints eliminate most charges, monitor S3 CRR costs |
| **Availability** | RDS primary failure during peak hours | High | Low | **MEDIUM** | Multi-AZ enabled, automatic failover in 60-120 seconds, test monthly |
| **Availability** | Complete regional failure (Singapore) | Critical | Very Low | **HIGH** | Documented DR runbook, quarterly DR drills, maintain S3 CRR to us-east-1 |
| **Availability** | All ECS tasks fail after bad deployment | High | Medium | **HIGH** | Implement blue/green deployments, automated rollback, thorough testing in staging |
| **Performance** | ElastiCache eviction under load | Medium | Medium | **MEDIUM** | Monitor hit rate (target >80%), increase instance size if needed |
| **Performance** | Database connection pool exhaustion | Medium | Medium | **MEDIUM** | Set max connections to 20 per task, monitor with Performance Insights |
| **Security** | Exposed RDS endpoint to internet | Critical | Low | **CRITICAL** | Security Group restricts to ECS tasks only, no public access, quarterly audit |
| **Security** | Leaked IAM credentials in code | Critical | Low | **CRITICAL** | Use IAM roles exclusively, never hardcode secrets, automated scanning with git-secrets |
| **Security** | S3 bucket misconfiguration (public access) | High | Low | **HIGH** | OAC enforced, S3 Block Public Access enabled, quarterly review |
| **Data** | Accidental database deletion | High | Very Low | **MEDIUM** | Enable RDS deletion protection, require MFA for destructive operations |
| **Data** | Data loss during regional failure | Medium | Very Low | **LOW** | RDS automated backups (7-day retention), test restore process monthly |
| **Operational** | Failed deployment with no rollback plan | Medium | Medium | **MEDIUM** | Document rollback procedure, keep previous 3 task definition revisions |

#### **7.2. Detailed Mitigation Plans**

**Cost Control Measures:**

*Proactive Monitoring:*

```
Daily Checks:
  - AWS Cost Explorer: Current spend vs forecast
  - ECS Service: Task count = expected (2 per service)
  - CloudWatch Billing: No unexpected alarms
  - S3 metrics: CRR data transfer within normal range

Weekly Reviews:
  - Top 5 cost drivers analysis
  - Comparison to previous week
  - Free Tier usage tracking
  - Forecast adjustment for end of month

Monthly Actions:
  - Generate detailed cost report
  - Review and optimize resource allocation
  - Update budget alerts for next month
  - Document lessons learned

```

*Emergency Cost Reduction Plan (Execute if >100% budget):*

| **Action** | **Time to Execute** | **Monthly Savings** | **Impact** |
| --- | --- | --- | --- |
| 1\. Reduce ECS tasks to 1 per service | 5 minutes | $18 | Reduced redundancy, acceptable for emergency |
| 2\. Stop ElastiCache cluster temporarily | 2 minutes | $12 | Slower performance, users may notice |
| 3\. Disable S3 CRR temporarily | 5 minutes | $1 | No new DR backups, existing data safe |
| 4\. Reduce CloudWatch log retention to 1 day | 2 minutes | $2 | Limited debugging history |
| 5\. Scale down to db.t3.micro Single-AZ (risky) | 10 minutes | $7.50 | **High risk of downtime, emergency only** |
| **Total Potential Savings:** | **24 minutes** | **$40.50** | Acceptable for 1-2 weeks while investigating |

**Security Hardening:**

*Network Security Configuration:*

```
Security Group: ALB-SG
  Inbound:
    - Port 443 (HTTPS) from 0.0.0.0/0
    - Port 80 (HTTP) from 0.0.0.0/0 (redirect to 443)
  Outbound:
    - All traffic to ECS-Tasks-SG

Security Group: ECS-Tasks-SG
  Inbound:
    - Ports 8080, 8081, 8082, 9998, 9999 from ALB-SG only
    - Ports 8080, 8081, 8082, 9998, 9999 from ECS-Tasks-SG (inter-service communication)
  Outbound:
    - Port 3306 to RDS-SG
    - Port 6379 to Redis-SG
    - Port 443 to VPC Endpoints (S3, ECR, CloudWatch)

Security Group: RDS-SG
  Inbound:
    - Port 3306 from ECS-Tasks-SG only
  Outbound:
    - None (no outbound connections)

Security Group: Redis-SG
  Inbound:
    - Port 6379 from ECS-Tasks-SG only
  Outbound:
    - None

```

*IAM Roles Configuration:*

```
Task Execution Role (ECS infrastructure):
  Permissions:
    - ecr:GetDownloadUrlForLayer
    - ecr:BatchGetImage
    - logs:CreateLogStream
    - logs:PutLogEvents

Task Role (Application permissions):
  Permissions:
    - s3:GetObject on sgutodolist-assets-sg/*
    - s3:PutObject on sgutodolist-assets-sg/uploads/*
    - ses:SendEmail for notification service
    - cloudwatch:PutMetricData for custom metrics

```

*Security Audit Checklist (Monthly):*

-   [ ] Review all Security Group rules for unnecessary open ports

-   [ ] Verify RDS has no public accessibility

-   [ ] Confirm S3 Block Public Access is enabled

-   [ ] Check IAM roles for over-permissive policies

-   [ ] Review CloudTrail logs for suspicious API calls

-   [ ] Verify all data at rest encryption (RDS, S3)

-   [ ] Confirm SSL/TLS for all data in transit

-   [ ] Rotate RDS master password (quarterly)

-   [ ] Review and update security group descriptions

**Performance Optimization:**

*Caching Strategy:*

| **Data Type** | **Cache Location** | **TTL** | **Eviction Policy** | **Monitoring Metric** |
| --- | --- | --- | --- | --- |
| User sessions | Redis | 30 minutes | TTL-based | Session count, hit rate >90% |
| User profiles | Redis | 5 minutes | LRU | Hit rate >85% |
| Board metadata | Redis | 10 minutes | LRU | Hit rate >80% |
| Task lists | Redis | 2 minutes | LRU | Hit rate >75% |
| Static reference data | Redis | 1 hour | TTL-based | Hit rate >95% |

*Database Query Optimization:*

```
-- Add indexes for common queries
CREATE INDEX idx_tasks_board_id ON tasks(board_id);
CREATE INDEX idx_tasks_assignee_id ON tasks(assignee_id);
CREATE INDEX idx_tasks_status ON tasks(status);
CREATE INDEX idx_boards_user_id ON boards(user_id);
CREATE INDEX idx_board_members_user_id ON board_members(user_id);

-- Monitor slow queries (>500ms)
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 0.5;

```

*Application-Level Optimizations:*

-   **Pagination:** Max 100 items per page, default 20

-   **Lazy Loading:** Use JPA `fetch = FetchType.LAZY` for relationships

-   **Connection Pooling:** HikariCP with max 20 connections per task

-   **Response Compression:** gzip enabled on ALB (automatic)

-   **API Rate Limiting:** 100 requests/minute per user (prevent abuse)

**Disaster Recovery Testing:**

*Monthly DR Drill (30 minutes):*

| **Test** | **Procedure** | **Expected Result** | **Pass/Fail** |
| --- | --- | --- | --- |
| 1\. ECS Task Failure | Manually stop one task | ALB routes traffic to healthy task, new task launches in 2-3 min |   |
| 2\. Cache Failure | Restart Redis cluster | App reconnects automatically, regenerates cache |   |
| 3\. Deployment Rollback | Deploy bad task definition, then rollback | Service returns to previous version in 3-5 min |   |
| 4\. RDS Snapshot Restore | Restore snapshot to new instance | Database accessible, data integrity verified |   |

*Quarterly Full DR Exercise (2 hours):*

1.  **Simulate Complete Regional Failure**

    -   Mark all Singapore resources as "unavailable"

    -   Document current RTO/RPO baselines

2.  **Execute DR Runbook:**

    ```
    Step 1: Restore RDS snapshot to us-east-1 (30 minutes)
    Step 2: Create ECS cluster in us-east-1 (10 minutes)
    Step 3: Deploy all 5 services with updated DB endpoint (20 minutes)
    Step 4: Create and configure ALB in us-east-1 (15 minutes)
    Step 5: Update Route 53 to point to new ALB (5 minutes)
    Step 6: Update CloudFront origin for API calls (10 minutes)
    Step 7: End-to-end testing (20 minutes)

    ```

3.  **Verify Complete Functionality:**

    -   User can log in and access their boards

    -   Tasks can be created, updated, deleted

    -   File uploads work correctly

    -   Notifications are sent

4.  **Document Findings:**

    -   Actual RTO achieved vs target (2-4 hours)

    -   Issues encountered and resolutions

    -   Updates needed to DR runbook

    -   Cost of maintaining DR environment

* * * * *

### **8\. Expected Outcomes**

#### **8.1. Technical Achievements**

**Performance Metrics:**

| **Metric** | **Target** | **Measurement Method** | **Baseline** |
| --- | --- | --- | --- |
| API Response Time (p95) | <100ms | CloudWatch custom metrics | 80-120ms |
| API Response Time (p99) | <300ms | CloudWatch custom metrics | 250-400ms |
| Page Load Time | <2 seconds | CloudFront + browser metrics | 1.5-2.5s |
| Cache Hit Rate | >80% | Redis INFO stats | 75-85% |
| Database Query Time (p95) | <50ms | RDS Performance Insights | 30-70ms |
| Availability (Monthly) | 99.5% | CloudWatch uptime monitoring | 99.4-99.7% |
| ECS Task Health | >95% | ALB target health checks | 97-99% |

**Scalability Capabilities:**

-   **Current Capacity:** 100-500 concurrent users with 10 ECS tasks (2 per service)

-   **Maximum Capacity (Same Cost):** 1,000 users with optimized caching and query performance

-   **Scaling Path:** Add tasks linearly (3-4 per service for 2,000+ users, ~$30/month additional)

**Operational Improvements:**

| **Before (Traditional EC2)** | **After (ECS Fargate)** | **Improvement** |
| --- | --- | --- |
| 4-6 hours/week server management | 30 min/week monitoring | **85% time savings** |
| Manual scaling decisions | Automatic target tracking | **Zero manual intervention** |
| 5-10 minute deployments | 3-5 minute rolling updates | **50% faster** |
| SSH access required | No server access needed | **Improved security** |
| Complex AMI management | Simple Docker images | **Easier version control** |

#### **8.2. Long-term Value**

**Skills Development:**

*Cloud Architecture:*

-   Hands-on experience with AWS core services (ECS, Fargate, RDS, ElastiCache, VPC)

-   Understanding of High Availability patterns (Multi-AZ, health checks, auto-scaling)

-   Cost optimization strategies for cloud infrastructure

-   Trade-off analysis: Performance vs Cost vs Availability

*Container & Microservices:*

-   Docker containerization best practices

-   Microservices communication patterns (service discovery, API gateways)

-   Container orchestration with ECS Fargate

-   Service mesh concepts (Cloud Map for DNS-based discovery)

*DevOps Practices:*

-   CI/CD pipeline design and implementation

-   Infrastructure monitoring and observability

-   Incident response and disaster recovery

-   Rolling deployments and zero-downtime updates

*Security:*

-   Least-privilege IAM policies

-   Network segmentation with Security Groups

-   Encryption at rest (RDS, S3) and in transit (TLS/SSL)

-   Security audit and compliance practices

**Portfolio Project Value:**

*Demonstrates to Employers:*

-   ✅ Production-ready system design (not just tutorials)

-   ✅ Cost consciousness and budget management

-   ✅ Modern cloud-native architecture patterns

-   ✅ End-to-end project delivery (planning → implementation → testing → documentation)

-   ✅ Ability to work within constraints (budget, technology, time)

*Interview Talking Points:*

-   "Reduced operational costs by 40% while maintaining 99.5% uptime using ECS Fargate"

-   "Implemented Multi-AZ disaster recovery with <2 minute RTO for AZ failures"

-   "Designed microservices architecture serving 500+ concurrent users on $30/month budget"

-   "Achieved <100ms API response times through strategic caching and query optimization"

**Business Foundation:**

*Monetization Potential:*

-   Current architecture supports 100-500 users at $30/month

-   Revenue model: $5/user/month = $500-2,500/month potential

-   Break-even: 6-10 paying users

-   Profit margin: 97-99% after break-even

*Scaling Roadmap:*

| **Users** | **Monthly Cost** | **Revenue ($5/user)** | **Profit** | **Required Changes** |
| --- | --- | --- | --- | --- |
| 100 | $30 | $500 | $470 | None (current architecture) |
| 500 | $50 | $2,500 | $2,450 | Increase tasks to 3 per service |
| 1,000 | $80 | $5,000 | $4,920 | Upgrade RDS to db.t3.small, add CloudWatch dashboards |
| 5,000 | $300 | $25,000 | $24,700 | Multi-region (add ap-southeast-2), Aurora RDS, managed Redis |
| 10,000+ | $800+ | $50,000+ | $49,200+ | Multi-region active-active, Aurora Global DB, ECS auto-scaling |

**Career Impact:**

*AWS Certification Alignment:*

-   **Cloud Practitioner:** Covers 70% of services used (EC2, RDS, S3, CloudFront)

-   **Solutions Architect Associate:** Direct experience with 80% of exam topics (VPC, IAM, HA patterns)

-   **SysOps Administrator:** Hands-on with monitoring, scaling, cost optimization

*Open Source Contribution:*

-   Template repository for "Single-Region HA SaaS on AWS Free Tier"

-   Blog series: "Building a Production SaaS for $30/month"

-   Conference talk: "Cost-Optimized Cloud Architecture Patterns"

*Next Project Ideas (Building on This Foundation):*

-   Add real-time collaboration with WebSockets (AWS API Gateway WebSocket)

-   Implement full-text search with Amazon OpenSearch

-   Build AI-powered task recommendations with Amazon Bedrock

-   Add mobile app with AWS Amplify and GraphQL API

* * * * *

### **9\. Future Development Roadmap**

The current Single-Region HA architecture provides a cost-optimized, production-ready foundation. As the platform grows in users and revenue, the following enhancements can be implemented incrementally.

#### **9.1. Phase 2: Multi-Region Expansion (6-12 months)**

**When to Consider:**

-   User base exceeds 1,000 active users

-   Significant user population outside Southeast Asia (>20%)

-   Monthly revenue justifies additional infrastructure cost ($300+)

-   Business requires <50ms API latency globally

**Target Architecture:**

| **Component** | **Current (Phase 1)** | **Future (Phase 2)** |
| --- | --- | --- |
| **Regions** | ap-southeast-1 (Singapore) | + ap-southeast-2 (Sydney), us-west-2 (Oregon) |
| **Routing** | Route 53 (single endpoint) | Route 53 Latency-Based Routing with health checks |
| **Database** | RDS Multi-AZ (single region) | Amazon Aurora Global Database (multi-region) |
| **Compute** | ECS Fargate (Singapore only) | ECS Fargate clusters in each region |
| **Caching** | ElastiCache (Singapore) | Regional ElastiCache clusters + Global Datastore |
| **Storage** | S3 CRR (passive DR) | S3 Multi-Region Access Points (active-active) |
| **Estimated Cost** | $30-70/month | $200-300/month |

**Benefits:**

-   ✅ True global low latency (<50ms for 95% of users)

-   ✅ Enhanced disaster recovery (automatic failover between regions)

-   ✅ Geographic compliance (data residency for EU, APAC, US)

-   ✅ Improved read performance (local read replicas)

#### **9.2. Phase 3: Enterprise Features (12-18 months)**

**Advanced Security:**

-   AWS Cognito for SSO/SAML integration

-   AWS WAF for API protection against attacks

-   AWS Shield Standard for DDoS protection

-   AWS GuardDuty for threat detection

**Observability & Analytics:**

-   AWS X-Ray for distributed tracing

-   Amazon OpenSearch for log analytics

-   Custom CloudWatch Dashboards for business metrics

-   AWS Cost Explorer API for automated cost reporting

**Performance Optimization:**

-   Amazon ElastiCache for Redis with Cluster Mode (horizontal scaling)

-   Amazon Aurora Serverless v2 (auto-scaling database)

-   AWS Global Accelerator for improved network performance

-   CloudFront Functions for edge computing

**Operational Excellence:**

-   AWS Systems Manager for parameter management

-   AWS Secrets Manager for credential rotation

-   Infrastructure as Code with Terraform or CDK

-   Automated DR testing with AWS Backup

#### **9.3. Phase 4: AI/ML Integration (18-24 months)**

-   Amazon Bedrock for AI-powered task recommendations

-   Amazon SageMaker for predictive analytics (task completion time)

-   Amazon Comprehend for sentiment analysis on comments

-   Amazon Rekognition for smart image tagging in attachments

* * * * *

### **10\. Conclusion**

This Cost-Optimized SaaS Task Management Platform demonstrates that production-grade applications can be built within AWS Free Tier constraints without sacrificing quality or reliability.

**Key Achievements:**

-   ✅ **99.5% availability** with Multi-AZ deployment

-   ✅ **<$30/month** operational cost in first month

-   ✅ **Zero server management** with ECS Fargate

-   ✅ **<100ms API response times** with strategic caching

-   ✅ **Disaster recovery** with S3 Cross-Region Replication

**Competitive Advantages:**

-   Modern cloud-native architecture ready for enterprise scale

-   Comprehensive documentation and operational runbooks

-   Clear scaling path from 100 to 100,000+ users

-   Foundation for future AI/ML features

**Next Steps:**

1.  Complete Phase 1 implementation (4-6 weeks)

2.  Deploy to production and gather metrics (2-3 months)

3.  Document lessons learned and optimize based on real usage

4.  Evaluate Phase 2 expansion based on user growth and feedback

This architecture provides a solid foundation for learning, career development, and potential monetization while maintaining strict cost discipline and operational excellence.