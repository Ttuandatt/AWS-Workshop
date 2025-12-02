+++ 
title = "Proposal"
weight = 2
chapter = true
pre = "<b>2. </b>" 
+++

# Multi-Region SaaS Task Management Platform  
## A Global AWS Solution for Low-Latency, Resilient Collaboration  


### **1. Executive Summary**  
The Multi-Region SaaS Task Management Platform is designed to deliver a Trello/Asana-like experience with low latency, high availability, and regional scalability across Asia-Pacific.

Built with Spring Boot microservices on EC2 Auto Scaling Groups and RDS MySQL Multi-Region setup, the platform ensures real-time collaboration, task/board management, and cross-region resilience.

By leveraging AWS global infrastructure, the system provides:  

- Low-latency access through RDS Read Replicas and regional compute
- High availability via multi-AZ deployment and cross-region failover
- Global content delivery using S3 Multi-Region Access Points and CloudFront
- Cost-optimized architecture suitable for Free Tier constraints

The result is a platform where users in Southeast Asia and Australia collaborate seamlessly — even during regional outages — while maintaining operational efficiency and cost predictability.

### **2. Problem Statement** 

#### **2.1. What’s the Problem?** 
Traditional SaaS task management platforms typically operate within a single AWS region, which can lead to several limitations:

- High Latency: Users located far from the primary region experience slow response times and poor user experience
- Single Point of Failure: Regional outages can cause complete service disruption
- Limited Scalability: Geographic expansion requires significant infrastructure changes
- Poor Disaster Recovery: No automated failover mechanisms for business continuity

#### **2.2. The Solution** 
This project introduces a multi-region architecture using AWS global services.  

Key highlights:  
- **RDS MySQL with Cross-Region Read Replicas** – Primary writer in Singapore (ap-southeast-1) with read replica in Sydney (ap-southeast-2)
- **EC2 Auto Scaling Groups** per region for scalable, resilient compute
- **Route 53** latency-based routing for intelligent traffic distribution to the nearest region
- **S3 Multi-Region Access Points + Cross-Region Replication (CRR)** for global file delivery
- Regional caching with **ElastiCache Redis** for session management and hot data
- **Application Load Balancers** for high availability and health checks

This architecture achieves **resilience, performance, and scalability** while optimizing for **cost-effectiveness** within AWS Free Tier constraints.

#### **2.3. Benefits and ROI**  
- Reduced latency for users across Southeast Asia and Australia (20-30ms between regions)
- High availability with multi-AZ deployment and automated failover
- Cost-efficient design optimized for Free Tier and small-scale production
- Scalable foundation ready for enterprise growth and additional regions
- Business continuity with cross-region disaster recovery capabilities

### **3. Solution Architecture**  

{{< figurecaption src="/images/todolist-architecture-4.jpg" caption="Figure 1. Architecture solution" >}}

#### **3.1. Architecture Overview**

**Primary Region: ap-southeast-1 (Singapore)**
  - RDS MySQL Writer (db.t3.micro) with Multi-AZ
  - EC2 Auto Scaling Group (t3.micro instances)
  - ElastiCache Redis (t3.micro) for session caching
  - Application Load Balancer
  - S3 bucket for primary storage
  - VPC with public/private subnets, NAT Gateway, Internet Gateway

**Secondary Region: ap-southeast-2 (Sydney)**
  - RDS Read Replica (db.t3.micro)
  - EC2 Auto Scaling Group (t3.micro instances)
  - ElastiCache Redis (t3.micro) for session caching
  - Application Load Balancer
  - S3 bucket (replica via CRR)
  - VPC with public/private subnets, NAT Gateway, Internet Gateway

**Global Services:**
  - Route 53 for DNS and latency-based routing
  - S3 Multi-Region Access Points
  - CloudFront CDN for static asset delivery
  - API Gateway (optional) for API management

#### **3.2. Microservices Architecture**

The platform consists of the following Spring Boot microservices:
- Auth Service - User authentication, JWT token management
- Board Service - Board creation, management, permissions
- Task Service - Task CRUD operations, assignments, status updates
- Notification Service - Real-time notifications, event handling
- User Service - User profile management, preferences

Each service is:
- Containerized using Docker
- Deployed on EC2 instances via Auto Scaling Groups
- Load-balanced via Application Load Balancer
- Connected to RDS MySQL (writes to primary, reads from local replica)
- Caching frequently accessed data in ElastiCache Redis


#### **3.3. AWS Services Used**
|    Category   |                           Services                         |                        Purpose                      |
|---------------|------------------------------------------------------------|-----------------------------------------------------|
| Compute       | EC2 Auto Scaling Groups                                    | Scalable microservices hosting across regions       |
| Database      | Amazon RDS (MySQL/PostgreSQL)                              | Relational database with Multi-AZ and Read Replicas |
| Caching       | ElastiCache Redis                                          | Session management & hot data caching per region    |
| Storage       | Amazon S3 + Multi-Region Access Points                     | Object storage with cross-region replication        |
| Networking    | VPC, Internet Gateway, NAT Gateway, ALB                    | Global DNS routing and API management               |
| DNS & Routing | Amazon Route 53, API Gateway                               | Build, deploy, and replicate container images       |
| Security      | Security Groups, IAM                                       | Network security and access control                 |
| Observability | CloudWatch                                                 | Logging, metrics, monitoring & alarms               |




### **4. Service Roles Overview**

| AWS Service | Role in Architecture |
|-------------|--------------------|
| Route 53 | DNS routing users to nearest region based on latency with health check failover |
| API Gateway | API management layer with throttling, request validation and regional endpoints |
| VPC | Isolated virtual network (10.0.0.0/16 for ap-southeast-1, 10.1.0.0/16 for ap-southeast-2) |
| Public Subnet | Hosts Application Load Balancer with internet access via Internet Gateway |
| Private Subnet | Hosts EC2 instances, RDS, and ElastiCache without direct internet access |
| Internet Gateway | Enables public subnet resources (ALB) to communicate with the internet |
| NAT Gateway | Allows private subnet resources (EC2, RDS) to access internet securely for updates |
| Application Load Balancer | Distributes incoming traffic across EC2 instances with health checks and SSL termination |
| EC2 Auto Scaling Groups | Automatically scales Spring Boot microservices (t3.micro) based on CPU/traffic demand |
| ElastiCache for Redis | In-memory caching for sessions, API responses, and frequently accessed data per region |
| Amazon RDS | Primary database writer in Singapore with async read replica in Sydney |
| S3 Buckets | Object storage for user files, attachments, and static assets per region |
| S3 Multi-Region Access Points | Unified global endpoint for accessing S3 objects with automatic routing to nearest region |
| S3 Cross-Region Replication | Automatically replicates objects from Singapore S3 bucket to Sydney for redundancy |
| Security Groups | Stateful firewall rules controlling inbound/outbound traffic for EC2, RDS, ALB, ElastiCache |
| IAM Roles | Secure service-to-service authentication without hardcoded credentials |
| CloudWatch | Centralized monitoring, logging, custom metrics, dashboards and alarms |



### **5. Service Flow**

#### **5.1. User Request Flow**

Read Operations (90% of traffic):

1. User in Singapore accesses app.taskmanager.com
2. Route 53 resolves to ap-southeast-1 (Singapore) based on latency
3. CloudFront serves static assets (CSS, JS, images) from edge location
4. API request → ALB (ap-southeast-1) → EC2 Auto Scaling Group
5. Spring Boot service checks ElastiCache Redis for cached data
   - Cache HIT → Return immediately (latency < 5ms)
   - Cache MISS → Query RDS MySQL Writer (local read, latency 5-10ms)
6. Response returned to user

User in Sydney/Australia:
1. Route 53 resolves to ap-southeast-2 (Sydney) based on latency
2. CloudFront serves static assets from Sydney edge location
3. API request → ALB (ap-southeast-2) → EC2 Auto Scaling Group
4. Spring Boot service checks local ElastiCache Redis
   - Cache HIT → Return immediately
   - Cache MISS → Query RDS Read Replica (local read, latency 5-10ms)
5. Response returned to user


Write Operations (10% of traffic):

**User in Singapore:**
1. Write request → ALB (ap-southeast-1) → EC2 → RDS Writer
2. Data committed to primary database (latency 10-20ms)
3. Async replication to Sydney Read Replica (5-30 seconds lag)
4. ElastiCache invalidated/updated in both regions via pub/sub
5. Success response to user

**User in Sydney:**
1. Write request → ALB (ap-southeast-2) → EC2
2. Spring Boot forwards write to ap-southeast-1 RDS Writer
3. Cross-region write (latency 50-100ms, acceptable for writes)
4. Async replication back to Sydney Read Replica
5. Cache invalidation via EventBridge + Lambda
6. Success response to user

#### **5.2. Developer CI/CD Flow**
Development & Deployment Workflow:

1. Developer commits code to Git repository
2. Build Spring Boot application (JAR or Docker image)
3. Upload artifacts to S3 bucket in ap-southeast-1
4. S3 Cross-Region Replication automatically copies to ap-southeast-2
5. Update EC2 Auto Scaling Group launch template with new AMI/image
6. Perform rolling deployment:
   - Launch new EC2 instances with updated code
   - ALB performs health checks
   - Gradually shift traffic from old to new instances
   - Terminate old instances after successful deployment
7. CloudWatch monitors deployment metrics and application health
8. Rollback mechanism: Keep previous version in S3 for quick revert

**Note:** CI/CD services like CodePipeline can be added later for full automation. For initial implementation, focus on manual deployment process.


#### **5.3. Data & HA/DR Flow**

Data Replication Strategy:

| Data Type         | Replication Method                  | Latency         | Purpose                          |
|------------------|-----------------------------------|----------------|---------------------------------|
| Transactional Data | RDS async replication             | 5-30 seconds   | User data, boards, tasks, comments |
| Session Data      | ElastiCache Redis (per region)    | N/A (regional) | User sessions, temporary auth tokens |
| Static Assets     | S3 Cross-Region Replication       | Minutes        | Images, attachments, frontend files |
| Application Code  | S3 CRR for artifacts              | Minutes        | Spring Boot JARs, Docker images |


**Failover Scenarios:**

**Scenario 1: EC2 Instance Failure**
1. ALB health check detects unhealthy EC2 instance
2. ALB stops routing traffic to failed instance
3. Auto Scaling Group launches replacement instance
4. New instance passes health check and receives traffic

Duration: 2-3 minutes | Impact: None (other instances handle load)


**Scenario 2: Availability Zone Failure**
1. All EC2 instances in one AZ fail
2. ALB routes all traffic to healthy AZ (if Multi-AZ configured)
3. Auto Scaling Group launches replacement capacity
4. RDS Multi-AZ automatically fails over to standby (if configured)

Duration: 5-10 minutes | Impact: Possible brief latency spike


**Scenario 3: Regional Failure (Singapore)**
1. Route 53 health checks detect region failure
2. DNS failover automatically routes to ap-southeast-2 (Sydney)
3. Manual RDS promotion: Promote Sydney Read Replica to Writer
4. Update application configuration to point to new writer endpoint
5. All traffic now served from Sydney region

Duration: 15-30 minutes | Impact: Read-only mode until promotion



### **6. Budget Estimation**


**Minimal Setup (Recommended for Free Tier)**

| AWS Service          | Cost (per month) | Notes |
|---------------------|----------------|-------|
| EC2 (t3.micro)       | $0.00          | 750 hours Free Tier, 4 instances × 12h/day = 720h/month |
| RDS MySQL (db.t3.micro) | $15.00       | Writer only, 744 hours - exceeds Free Tier |
| RDS Read Replica (db.t3.micro) | $15.00 | Sydney region, 744 hours |
| S3 Standard          | $2.00          | 10 GB storage, cross-region replication |
| Route 53             | $1.00          | 1 hosted zone, basic queries |
| CloudWatch           | $3.00          | Basic metrics, 3-day log retention |
| Data Transfer        | $5.00          | Cross-region + internet egress |
| VPC                  | $0.00          | VPC itself is free, NAT Gateway excluded |
| **Total**            | **$41.00**     | $492/year |




### **7. Risk Assessment**

**Risk Matrix**

| Risk                         | Impact | Probability| Priority|
|------------------------------|--------|------------|---------|
| RDS Replication Lag          | Medium | Medium     | High    |
| Cross-Region Write Latency   | Medium | High       | Medium  |
| NAT Gateway SPOF             | High   | Low        | Medium  |
| Cost Overruns                | High   | High       | Critical|
| Free Tier Exhaustion         | Medium | High       | High    |
| Manual Failover Complexity   | High   | Medium     | High    |
| Security Vulnerabilities     | High   | Medium     | Critical|
| Data Consistency Issues      | High   | Low        | Medium  |



**Mitigation Strategies**

**Network & Infrastructure:**

RDS Replication: Monitor replication lag with CloudWatch alarms (<60s threshold). Implement application-level checks for critical writes.
NAT Gateway: Deploy in multiple Availability Zones. Consider VPC endpoints for S3/CloudWatch to reduce NAT dependency.
Auto Scaling: Pre-warm instances during known traffic peaks. Optimize Spring Boot startup time.

**Cost Management:**

AWS Budgets: Set up alerts at 80%, 90%, 100% of monthly budget.
Resource Scheduling: Stop non-essential resources during off-hours (weeknights, weekends).
Right-sizing: Start small (t3.micro), scale up only when metrics justify it.
Free Tier Monitoring: Track usage daily via Cost Explorer to stay within 750-hour limits.

**Security:**

Security Groups: Follow least-privilege principle, review quarterly.
CloudTrail: Enable logging for all API calls, retain for 90 days minimum.
Regular Audits: Weekly log reviews, monthly security assessments.
Secrets Management: Use environment variables, rotate credentials regularly.

**Contingency Plans**
**Scenario 1: Regional Failure (Singapore)**

Route 53 automatically redirects traffic to Sydney (5-10 minutes).
Manually promote Sydney RDS Read Replica to Writer (10-20 minutes).
Update application configuration to use new database endpoint.
Total RTO: ~30 minutes, RPO: <1 minute.

**Scenario 2: Budget Exceeded**

Identify cost spike via AWS Cost Explorer.
Stop non-essential resources (dev/test EC2, unused snapshots).
Reduce Auto Scaling maximum capacity temporarily.
Optimize database queries to reduce RDS load.
Consider reverting to manual deployment if CI/CD costs spike.

**Scenario 3: Replication Lag >5 Minutes**

Temporarily direct critical reads to primary database.
Investigate cause (high write volume, network issues, large transactions).
Implement aggressive caching to reduce database load.
Consider scaling up RDS instance class if CPU-bound.




### **8. Expected Outcomes**

#### **8.1. Technical Improvements**

**Performance Gains:**

- Latency Reduction: 80% improvement for Sydney users (from 200ms to 20ms for reads).
- Availability: 99.5%+ uptime vs 95% with single-region architecture.
- Scalability: Support 1,000+ concurrent users with current infrastructure.
- Response Time: <50ms for cached data, <100ms for database queries.

**Operational Capabilities:**

Real-time monitoring and alerting via CloudWatch dashboards.
Automated scaling based on traffic patterns (2-10 instances per region).
Cross-region disaster recovery with <30 minute RTO.
Cost-optimized infrastructure using AWS Free Tier benefits.

**Architecture Foundation:**

Multi-region pattern replicable to other geographic areas (US, Europe).
Microservices architecture ready for containerization (ECS/EKS).
Scalable from 100 to 100,000+ users by upgrading instance sizes.
Production-ready security with VPC isolation, encryption, IAM roles.

#### **8.2. Long-term Value**

**Skills Development:**

- Hands-on experience with AWS core services (EC2, RDS, VPC, Route 53, S3).
- Understanding of multi-region architecture patterns and trade-offs.
- DevOps practices: Infrastructure as Code, CI/CD, monitoring, incident response.
- Cost optimization and cloud financial management expertise.

**Portfolio Project:**

- Demonstrates cloud architecture expertise to potential employers.
- Shows end-to-end project delivery: planning, implementation, testing, documentation.
- Proves ability to work within constraints (budget, time, technology).
- Real-world production-ready system (not just tutorial follow-along).

**Business Foundation:**

- Reusable architecture for future SaaS applications.
- 1-year operational data for analytics and AI/ML projects.
- Foundation for enterprise features (SSO, audit logging, multi-tenancy).
- Potential monetization path (charge per user/tenant).

**Career Impact:**

- AWS certification preparation (Cloud Practitioner, Solutions Architect, SysOps).
- Interview talking points for cloud/DevOps positions.
- Open-source contribution opportunity (template for multi-region apps).
- Foundation for technical blog posts and conference talks.
