+++
title = "Infrastructure & ALB Setup (RDS, Redis, Cloud Map, ALB Routing)"
weight = 2
chapter = false
pre = " <b> 5.3.2.2. </b> "
alwaysopen = true
+++


This phase provisions the data layer components, service discovery mechanism, and load balancing infrastructure.

### SSL Certificate Provisioning

**Request ACM Certificate**

1.  Switch region to **Singapore (ap-southeast-1)**
2.  Navigate to **Certificate Manager (ACM)**
3.  Select **Request a certificate** → **Request a public certificate**
4.  Configure certificate request:
    -   **Domain name**: `sgutodolist.com`
    -   **Validation method**: DNS validation
5.  Click **Request**
6.  Access the certificate details → Under **Domains** section → Click **Create records in Route 53**
7.  Wait for status to change to **Issued** (typically 5-10 minutes)

* * * * *

### RDS MYSQL (Primary Database)

*Goal: Create the main MySQL 8.0 Database located in Private Subnet (AZ A).*

1.  Go to **RDS** > **Create database**.

2.  **Choose a database creation method:** **Standard create** (Full Configuration).

3.  **Engine options:** Select **MySQL**.

4.  **Engine Version:** Select **8.0.x** (e.g., 8.0.35 or 8.0.39) to match Docker Compose.

5.  **Templates:** Select **Free tier**.

6.  **Settings:**

    -   **DB Instance identifier:** `sgu-todolist-db`

    -   **Master username:** `root`

    -   **Master password:** `12345678` (Example).

7.  **Instance configuration:** Select **`db.t3.micro`**.

8.  **Storage:**

    -   **Storage type:** `gp2` or `gp3`.

    -   **Allocated storage:** `20 GiB`.

9.  **Connectivity (IMPORTANT):**

    -   **Compute resource:** Don't connect to an EC2 compute resource.

    -   **VPC:** Select `SGU-Microservices-vpc`.

    -   **DB Subnet group:** Select `Create new` (or select the existing one pointing to Private Subnets).

    -   **Public access:** **NO** (Strictly Private).

    -   **VPC security group:** Select **`private-db-sg`** (Remove `default`).

    -   **Availability Zone:** Select **`ap-southeast-1a`**.

10. **Additional configuration:**

    -   **Initial database name:** (Leave blank, we will create it via Bastion later).

    -   **Backup:** **Enable automated backups** (⚠️ **Mandatory**: You must enable this to create a Read Replica later. Set retention period to **1 day** to save costs).

11. Click **Create database**.

*Wait until Status is **Available**, then copy the **Endpoint**.*

### RDS READ REPLICA (For High Availability Demo)

*Goal: Create a secondary database in a different Availability Zone (AZ B) to demonstrate Multi-AZ architecture.*

1.  In the **RDS Console** > **Databases**.

2.  Select the primary database: **`sgu-todolist-db`**.

3.  Click **Actions** button > Select **Create read replica**.

4.  **Settings:**

    -   **DB instance identifier:** `sgu-todolist-db-replica`.

5.  **Instance configuration:**

    -   **DB instance class:** Select **`db.t3.micro`** (Must match Primary).

6.  **Availability:**

    -   **Deployment options:** Select **Single-AZ DB instance deployment** (The Replica itself is single AZ, but combined with Primary, it forms a Multi-AZ system).

7.  **Connectivity (CRITICAL):**

    -   **AWS Region:** `Asia Pacific (Singapore)`.

    -   **Availability Zone:** Select **`ap-southeast-1b`** (Must be different from Primary).

    -   **Public access:** **Not publicly accessible**.

    -   **Existing VPC security groups:** Select **`private-db-sg`** (Remove `default`).

8.  **Monitoring:**

    -   **Enable Enhanced monitoring:** **Uncheck** (Save cost).

9.  Click **Create read replica**.

*Result: You will see two databases. One in `ap-southeast-1a` (Primary) and one in `ap-southeast-1b` (Replica).*

* * * * *

### ELASTICACHE REDIS

*Goal: Create a simple, cheap Redis `t3.micro`, without using Cluster mode.*

1. Go to **ElastiCache** > **Redis OSS caches** > **Create Redis OSS cache**.

2. **Cluster settings:** 

- Engine: **Redis OSS**. 

- Deployment option: **Node-based cluster** (This must be selected to adjust the configuration). 

- Creation method: **Cluster cache**.

3. **Cluster info:** 

- Cluster mode: **Disabled**. 

- Name: `sgu-redis`.

4. **Cache settings:** 

- Node type: **`cache.t3.micro`**. 

- Number of replicas: **0**.

5. **Connectivity:** 

- **Subnet groups:** Select `Create new` (if not already there). 

- Name: `sgu-redis-subnet-group`. 

- VPC: `SGU-Microservices-vpc`. 

- Subnets: Select 2 **Private Subnets**. 

- **VPC security groups:** Select **`private-db-sg`**. 

- **Availability Zone placements:** Select **Specify Availability Zones** -> **`ap-southeast-1a`**.

6. Click **Next**.

7. **Advanced settings:** 

- **Enable automatic backups:** **Uncheck** (Save money). 

- **Logs:** Disable all.

8. Scroll to the bottom and click **Create**.

*After creating, copy **Primary Endpoint**.*

* * * * *

### AWS CLOUD MAP (Service Discovery)

*Goal: Create an internal `.local` domain for Kafka and AI Service to talk to each other without a Load Balancer.*

1. Go to **Cloud Map** service.

2. Click **Create namespace**.

3. **Namespace name:** `sgu.local`.

4. **Description:** SGU Internal Network.

5. **Instance discovery:** Select **API calls and DNS queries in VPCs**.

6. **VPC:** Select `SGU-Microservices-vpc`.

7. Click **Create namespace**.

* * * * *

### ALB ROUTING (Target Groups & Load Balancer)

This is the most important routing part for users to access the system.

#### 4.1 Create 5 Target Groups (Repeat 5 times)

Go to **EC2** > **Target Groups** > **Create target group**.

**Common configuration for all 5:**

- **Target type:** **IP addresses** (Required for ECS Fargate).

- **Protocol:** **HTTP**.

- **IP address type:** **IPv4**.

- **VPC:** `SGU-Microservices-vpc`.

- **Health check path:** `/actuator/health`.

- *Note: In the next "Register targets" step, DO NOT enter any IP, click Create.*

**List of 5 Target Groups to create:**

| **Name** | **Port** | **Health Check Path** |
| --- | --- | --- |
| `auth-tg` | 9999 | `/api/auth/actuator/health` |
| `user-tg` | 8081 | `/api/user/actuator/health` |
| `task-tg` | 8082 | `/api/taskflow/actuator/health` |
| `noti-tg` | 9998 | `/api/notification/actuator/health` |

* * * * *

#### 4.2 Create Application Load Balancer (ALB)

Go to **EC2** > **Load Balancers** > **Create load balancer** > **Application Load Balancer**.

1. **Basic configuration:** 

- Name: `sgu-alb`. 

- Scheme: **Internet-facing**. 

- IP address type: **IPv4**.

2. **Network mapping:** 

- VPC: `SGU-Microservices-vpc`. 

- **Mappings:** Select 2 **Public Subnets** (For example `...public1...` and `...public2...`). ⚠️ *If you get this wrong, the whole thing will collapse.*

3. **Security groups:** 

- Select **`public-alb-sg`**.

4. **Listeners and routing (Create 2):** 

- **HTTP:80**: Forward to `api-gateway-tg`. 

- **HTTPS:443** (Click Add listener): 

- Default action: Forward to `api-gateway-tg`. 

- **Secure listener settings:** Select ACM certificate `sgutodolist.com`

**Important:** Để mọi service phải thông qua api-gateway thì phải set Listener rules của Protocol:Port `HTTPS:443` là:  `/api/auth/*` (vì auth thì có thể đi 1 luồng riêng) và `Default`
- EC2 > Load balancers > sgu-alb > HTTPS:443 listener
- Ở mục Listener rules xóa các Rule có điều kiện Path là `/api/<service>/*`


<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.1-Network & Security Preparation (VPC, SG)" %}}" style="text-decoration: none; font-weight: bold;">
⬅ STEP 1: Network & Security Preparation
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.3-Code Update & Image Build (Create new Docker Image)" %}}" style="text-decoration: none; font-weight: bold;">
STEP 3: Code Update & Image Build ➡
</a>
</div>