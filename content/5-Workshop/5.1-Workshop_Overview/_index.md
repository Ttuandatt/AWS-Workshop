+++
title = "Workshop Overview"
weight = 1
chapter = false
pre = " <b> 5.1.  </b> "
+++

Project Introduction
-----------------

**SGU TodoList** is a task management application built using a Microservices architecture on the AWS Cloud platform. The project was initially designed to be deployed as a Multi-Region SaaS model to ensure high availability and disaster recovery. However, due to budget constraints and AWS Free Tier limitations, the team optimized the architecture for a Single-Region Deployment with a Cross-Region Failover mechanism for the frontend.

Overall architecture
-----------------

### Backend Architecture (Single-Region: Singapore)
```
            ┌──────────────────────────────────────────────────────┐
            │                 Internet Users                       │
            └───────────────────────┬──────────────────────────────┘
                                    │
                                    ▼
                            ┌────────────────┐
                            │   Route 53     │ ◄── DNS Management
                            │ (Global DNS)   │
                            └───────┬────────┘
                                    │
                                    ▼
                        ┌──────────────────────┐
                        │  Application Load    │
                        │  Balancer (ALB)      │ ◄── SSL/TLS Termination
                        │  + ACM Certificate   │     (sgutodolist.com)
                        └──────────┬───────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    │              │              │
                    ▼              ▼              ▼
                ┌─────────┐  ┌─────────┐   ┌──────────┐
                │  Auth   │  │  User   │   │Taskflow  │  ◄── ECS Fargate
                │ Service │  │ Service │   │ Service  │      Microservices
                └────┬────┘  └────┬────┘   └────┬─────┘      (Public Subnets)
                     │            │             │
                     └────────────┼─────────────┘
                                  │
                         ┌────────┴────────┐
                         │   API Gateway   │ ◄── Central Entry Point
                         │   (Port 8080)   │     + CORS + Rate Limiting
                         └────────┬────────┘
                                  │
                      ┌───────────┼─────────────┐
                      │           │             │
                      ▼           ▼             ▼
              ┌────────────┐ ┌─────────┐  ┌──────────┐
              │Notification│ │  Kafka  │  │ AI Model │
              │  Service   │ │ Cluster │  │ Service  │
              └──────┬─────┘ └────┬────┘  └────┬─────┘
                     │            │            │
                     └────────────┼────────────┘
                                  │
                       ┌──────────┴──────────┐
                       │                     │
                       ▼                     ▼
                  ┌─────────┐          ┌───────────┐
                  │   RDS   │          │  Redis    │  ◄── Data Layer
                  │  MySQL  │          │ElastiCache│     (Private Subnets)
                  └─────────┘          └───────────┘
```

### Frontend Architecture (Multi-Region Failover)
```
            ┌─────────────────────────────────────────────────────────┐
            │                      Internet Users                     │
            └──────────────────────────┬──────────────────────────────┘
                                       │
                                       ▼
                               ┌──────────────┐
                               │   Route 53   │ ◄── DNS: sgutodolist.com
                               └────────┬─────┘
                                        │
                                        ▼
                               ┌────────────────┐
                               │  CloudFront    │ ◄── Global CDN
                               │  Distribution  │     SSL Certificate
                               └────────┬───────┘     Custom Domain
                                        │
                          ┌─────────────┴─────────────┐
                          │    Origin Group (HA)      │
                          │   ┌─────────────────┐     │
                          │   │ Primary Origin  │     │
                          │   │  S3 Singapore   │ ◄───┼── Main Region
                          │   └─────────────────┘     │
                          │           │               │
                          │   ┌───────▼───────────┐   │
                          │   │ Failover Origin   │   │
                          │   │  S3 N.Virginia    │ ◄─┼── Backup Region
                          │   └───────────────────┘   │
                          └───────────────────────────┘
                                       │
                            ┌──────────┴──────────┐
                            │ S3 Cross-Region     │
                            │ Replication (CRR)   │ ◄── Auto Sync
                            └─────────────────────┘
```

### Main components

### **1\. Backend Services (ECS Fargate)**

| Service               | Port  | Chức năng                                               | Dependencies       |
|-----------------------|-------|--------------------------------------------------------|------------------|
| API Gateway           | 8080  | - Điều phối routing<br>- CORS handling<br>- Rate limiting | Redis, All Services |
| Auth Service          | 9999  | - Xác thực JWT<br>- OAuth2 (Google)<br>- Token management | RDS, Redis, Kafka |
| User Service          | 8081  | - Quản lý user profile<br>- User CRUD                 | RDS, Redis, Kafka |
| Taskflow Service      | 8082  | - Quản lý tasks<br>- Workflows                        | RDS, Redis, Kafka |
| Notification Service  | 9998  | - WebSocket real-time<br>- Push notifications         | RDS, Redis, Kafka |
| AI Model Service      | 9997  | - Task prioritization<br>- Smart suggestions          | Python Flask      |

### **2\. Infrastructure Components**

#### **Networking**

-   **VPC**: `10.0.0.0/16` at Singapore (ap-southeast-1)
-   **Public Subnets (2 AZs)**: ECS Tasks, ALB, Bastion
-   **Private Subnets (2 AZs)**: RDS, Redis (data)
-   **Security Strategy**: Eliminated NAT to save cost (~$30/month)

#### **Data Storage**

-   **RDS MySQL**: Primary database, Free Tier (`db.t3.micro`)
-   **ElastiCache Redis**: Caching + Rate limiting (`cache.t3.micro`)
-   **Kafka (ECS)**: Event streaming, Service Discovery (`kafka.sgu.local`)

#### **Load Balancing & SSL**

-   **Application Load Balancer**: HTTPS termination, path-based routing
-   **ACM Certificate**: `sgutodolist.com`
-   **Target Groups**: Each service has their onw target group with health checks

#### **Service Discovery**

-   **AWS Cloud Map**: Namespace `sgu.local`
-   **Internal DNS**:
    -   `auth.sgu.local:9999`
    -   `user.sgu.local:8081`
    -   `taskflow.sgu.local:8082`
    -   `notification.sgu.local:9998`
    -   `ai-model.sgu.local:9997`
    -   `kafka.sgu.local:9092`

Cost & Optimization
--------------------

### **Important Architectural Decisions**

| Factors               | Initial Choice          | Final Choice               | Savings       |
|-----------------------|------------------------|-----------------------------|---------------|
| **NAT Gateway**       | 2 NAT (HA)             | ❌ Not used                | ~$60/month    |
| **ECS Compute**       | EC2 Instances          | ✅ Fargate Spot (0.5 vCPU) | ~$40/month    |
| **RDS**               | Multi-AZ               | ✅ Single-AZ Free Tier     | ~$30/month    |
| **Redis**             | Cluster Mode           | ✅ Single Node             | ~$20/month    |
| **Backend Multi-Region** | Active-Active       | ✅ Single Region           | ~$200/month   |
| **Frontend HA**       | Multi-Region Active    | ✅ Failover Only           | ~$50/month    |


### **Public Subnet Strategy for ECS**

Instead of using expensive NAT Gateways, all ECS Tasks are deployed on **Public Subnets** with **Public IP** enabled. This allows:

- ✅ Load Docker images from ECR over the Internet
- ✅ Outbound connectivity to AWS services
- ✅ Save 100% on NAT Gateway costs
- ⚠️ Trade-off: Need to manage Security Groups carefully

Domain & SSL Strategy
---------------------

### **Production Domains**

-   **Frontend**: `https://sgutodolist.com` (CloudFront + ACM us-east-1)
-   **Backend API**: `https://sgutodolist.com` (ALB + ACM ap-southeast-1)

### **SSL Certificates**

-   **Certificate 1** (us-east-1): For CloudFront (Must be in Virginia)
    -   Domain: `sgutodolist.com`, `*.sgutodolist.com`
-   **Certificate 2** (ap-southeast-1): For ALB Singapore
    -   Domain: `sgutodolist.com`

    
Security Architecture
---------------------

### **Security Groups Matrix**
| SG Name         | Purpose             | Inbound Rules                                 |
|-----------------|-------------------|-----------------------------------------------|
| `public-alb-sg` | ALB public facing  | 0.0.0.0/0:443, 0.0.0.0/0:80                  |
| `ecs-app-sg`    | ECS Tasks          | ALB:8080-8082, 9998-9999; Self: all ports (inter-service) |
| `private-db-sg` | RDS + Redis + Kafka| ecs-app-sg:3306, 6379, 9092                  |
| `bastion-sg`    | SSH Jump Host      | My IP:22                                      |


### **Authentication Flow**

```
User → CloudFront → API Gateway → Auth Service
                                      ↓
                              JWT + Redis Session
                                      ↓
                              Google OAuth2 (Optional)
```

Team Information
----------------

-   **Project**: SGU TodoList - Task Management System
-   **Architecture**: Single-Region Microservices (Cost-Optimized)
-   **Primary Region**: Asia Pacific (Singapore) - ap-southeast-1
-   **Failover Region**: US East (N. Virginia) - us-east-1 (Frontend only)
-   **Deployment Model**: AWS Free Tier Optimized