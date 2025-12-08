+++
title = "Network & Security Preparation (VPC, SG)"
weight = 1
chapter = false
pre = " <b> 5.3.2.1. </b> "
alwaysopen = true
+++

This phase establishes the foundational network infrastructure and security boundaries for the microservices deployment.

### VPC and Subnet Configuration

**Step 1: Create VPC Infrastructure**

1.  Navigate to **VPC Console** → **Create VPC**
2.  Select **VPC and more** option
3.  Configure VPC parameters:

| Parameter | Value | Rationale |
| --- | --- | --- |
| Name tag auto-generation | `SGU-Microservices` | Naming convention |
| IPv4 CIDR block | `10.0.0.0/16` | Standard private network range |
| Number of Availability Zones | 2 (ap-southeast-1a, ap-southeast-1b) | High availability across AZs |
| Number of public subnets | 2 | For internet-facing resources |
| Number of private subnets | 2 | For data layer isolation |
| NAT gateways | None | Cost optimization (~$30/month savings) |
| VPC endpoints | None | Cost optimization |
| DNS options | Enable DNS hostnames + Enable DNS resolution | Required for service discovery |

1.  Click **Create VPC**

**Network Architecture Result:**

```
VPC: 10.0.0.0/16
├── Public Subnet 1:  10.0.0.0/20  (ap-southeast-1a)
├── Public Subnet 2:  10.0.16.0/20 (ap-southeast-1b)
├── Private Subnet 1: 10.0.128.0/20 (ap-southeast-1a)
└── Private Subnet 2: 10.0.144.0/20 (ap-southeast-1b)
```

* * * * *

### Security Groups Configuration

Security groups act as virtual firewalls controlling inbound and outbound traffic for AWS resources.

**Step 2: Create Security Groups**

Navigate to **VPC** → **Security Groups** → **Create security group**. Create four security groups with the following specifications:

#### Security Group 1: public-alb-sg (Application Load Balancer)

| Parameter | Value |
| --- | --- |
| Name | `public-alb-sg` |
| Description | Security group for SGUTODOLIST ALB |
| VPC | `SGU-Microservices-VPC` |

**Inbound Rules:**

| Type | Protocol | Port | Source | Purpose |
| --- | --- | --- | --- | --- |
| HTTPS | TCP | 443 | 0.0.0.0/0 | Public HTTPS access |
| HTTP | TCP | 80 | 0.0.0.0/0 | Public HTTP access (redirect to HTTPS) |

* * * * *

#### Security Group 2: ecs-app-sg (ECS Application Containers)

| Parameter | Value |
| --- | --- |
| Name | `ecs-app-sg` |
| Description | Security group for SGUTODOLIST Service Container |
| VPC | `SGU-Microservices-VPC` |

**Inbound Rules - Phase 1 (ALB to Services):**

| Type | Protocol | Port | Source | Purpose |
| --- | --- | --- | --- | --- |
| Custom TCP | TCP | 8080 | public-alb-sg | ALB to API Gateway |
| Custom TCP | TCP | 8081 | public-alb-sg | ALB to User Service |
| Custom TCP | TCP | 8082 | public-alb-sg | ALB to Taskflow Service |
| Custom TCP | TCP | 9998 | public-alb-sg | ALB to Notification Service |
| Custom TCP | TCP | 9999 | public-alb-sg | ALB to Auth Service |
| Custom TCP | TCP | 9092 | public-alb-sg | Services call to Kafka |
| Custom TCP | TCP | 9997 | public-alb-sg | ALB to AI Service |

**Important:** Click **Create security group** before proceeding to Phase 2.

**Inbound Rules - Phase 2 (Inter-service Communication):**

1.  Select the newly created `ecs-app-sg`
2.  Edit inbound rules → Add rule
3.  Configure self-referencing rule:
    -   Type: `All TCP`
    -   Port range: `0-65535`
    -   Source: Select `ecs-app-sg` (self-reference)
    -   Purpose: Allow containers to communicate with each other

* * * * *

#### Security Group 3: private-db-sg (Data Layer)

| Parameter | Value |
| --- | --- |
| Name | `private-db-sg` |
| Description | Security group for SGUTODOLIST RDS & Redis & Kafka |
| VPC | `SGU-Microservices-VPC` |

**Inbound Rules:**

| Type | Protocol | Port | Source | Purpose |
| --- | --- | --- | --- | --- |
| MySQL/Aurora | TCP | 3306 | ecs-app-sg | RDS database access |
| Custom TCP | TCP | 6379 | ecs-app-sg | Redis cache access |
| Custom TCP | TCP | 9092 | ecs-app-sg | Kafka broker access |
| MySQL/Aurora | TCP | 3306 | bastion-sg | Access Database from the Bastion Host (Admin/Debug) |
| Custom TCP | TCP | 6379 | bastion-sg | Access Redis from the Bastion Host (Admin/Debug) |
| MySQL/Aurora | TCP | 3306 | 14.186.212.182/32 | Direct Access from a fixed IP address (Personal/Debug) |




* * * * *

#### Security Group 4: bastion-sg (SSH Jump Host)

| Parameter | Value |
| --- | --- |
| Name | `bastion-sg` |
| Description | Security group for SGUTODOLIST bastion |
| VPC | `SGU-Microservices-VPC` |

**Inbound Rules:**

| Type | Protocol | Port | Source | Purpose |
| --- | --- | --- | --- | --- |
| SSH | TCP | 22 | My IP | Secure shell access from operator's IP |

**Security Note:** Replace "My IP" with your actual public IP address for enhanced security.

* * * * *

### Network Validation Checklist

Before proceeding to the next phase, verify:

-   [ ]  VPC created with CIDR 10.0.0.0/16
-   [ ]  2 Public subnets across different AZs
-   [ ]  2 Private subnets across different AZs
-   [ ]  DNS hostnames and resolution enabled
-   [ ]  All 4 security groups created with correct rules
-   [ ]  `ecs-app-sg` includes self-referencing rule

* * * * *

<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="" style="text-decoration: none; font-weight: bold;">
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.2-Infrastructure & ALB Setup (RDS, Redis, Cloud Map, ALB Routing)" %}}" style="text-decoration: none; font-weight: bold;">
STEP 2: Infrastructure & ALB Setup ➡
</a>
</div>