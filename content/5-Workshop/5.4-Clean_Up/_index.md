+++
title = "Clean Up"
weight = 4
chapter = false
pre = " <b> 5.4.  </b> "
+++


Cleanup Guide
-----------------------------------------------------------

This guide provides a comprehensive instruction to completely terminate all deployed AWS resources across the **ap-southeast-1 (Singapore)** and **us-east-1 (N. Virginia)** regions. The sequence is optimized to avoid orphaned resources and subsequent charges.

### **I. Deployed AWS Services Summary**

Before proceeding, verify the existence of the following active services in your account:

| Service Category | Service/Resource Name | Region | Purpose |
| --- | --- | --- | --- |
| **Compute/Scaling** | ECS Cluster/ASG, EC2 Instances | ap-southeast-1 | Application Hosting |
| **Database/Caching** | RDS DB Instance (Multi-AZ), ElastiCache Redis Cluster | ap-southeast-1 | Data Persistence & Session Management |
| **Networking** | Application Load Balancer (ALB), Target Groups, NAT Gateway, VPC | ap-southeast-1 | Traffic Distribution & Internet Access |
| **Storage/Artifacts** | S3 Primary Bucket, S3 DR Bucket, ECR Repositories | ap-southeast-1, us-east-1 | File Storage & Image Hosting |
| **Global/DNS/Security** | CloudFront Distribution, ACM Certificates, Route 53 Hosted Zone | Global (Edge), us-east-1, ap-southeast-1 | CDN & SSL Security |


* * * * *

### **II. Cleanup Procedure (Step-by-Step)**

#### **1\. Clean Up Application Layer (ap-southeast-1)**

We start by eliminating resources that host the application and connect to the database.

1.  **Stop ECS/EC2 Compute:**

    -   **If using ECS:** 
        -   Go to **Amazon ECS** → **Clusters**. 
        -   Select Cluster → Tab **Services**. 
        -   Select all Services → Click **Update**. 
        -   Set **Desired tasks** to **0** → **Next** → **Update Service**. 
        -   Wait for tasks to stop. Then, select Services again → **Delete** → Confirm with `delete me`.


2.  **Delete Load Balancer (ALB) Components:**

    -   Go to **EC2** → **Target Groups**. Select Target Groups → Click **Actions** → **Delete**.

    -   Go to **EC2** → **Load Balancers**. Select ALB → Click **Actions** → **Delete**.

#### **2\. Delete Database and Caching Services**

1.  **Delete ElastiCache (Redis):**

    -   Go to **ElastiCache** → **Redis Clusters**. Select Cluster → Click **Actions** → **Delete**.

2.  **Delete RDS Database (CRITICAL COST STEP):**

    -   Go to **RDS** → **Databases**. Select Instance.

    -   If **Deletion protection** is *Enabled*, click **Modify** → Scroll to *Deletion protection* → Uncheck the box → **Continue** → **Apply immediately**.

    -   Select Instance → Click **Actions** → **Delete**.

    -   **Uncheck** **Create final snapshot** → Type the DB name to confirm → Click **Delete**.

#### **3\. Clean Up Global Services and Certificates**

These steps involve resources spanning multiple regions.

1.  **Delete CloudFront Distribution:**

    -   Go to **CloudFront**. Select Distribution → Click **Disable**. (Wait 10-15 minutes for status change).

    -   Once Disabled, select Distribution → Click **Delete**.

2.  **Delete ACM Certificates (Must be done in both regions):**

    -   **us-east-1 (N. Virginia):** Switch Region → Go to **ACM**. Select Certificate → Click **Delete**.

    -   **ap-southeast-1 (Singapore):** Switch Region → Go to **ACM**. Select Certificate → Click **Delete**.

#### **4\. Clean Up Storage and Artifacts**

1.  **Delete ECR Repositories:**

    -   Go to **Elastic Container Registry (ECR)**. Select Repositories → Click **Delete** → Type `delete` to confirm.

2.  **Clean and Delete S3 Buckets:**

    -   **Remove CRR:** Go to **S3** → Select Primary Bucket (`ap-southeast-1`) → Tab **Management** → **Replication Rules** → **Delete** the rule.

    -   **Empty:** For both S3 Buckets (ap-southeast-1 and us-east-1), go to Tab **Objects** → Select all → Click **Delete** → Type `permanently delete` to confirm.

    -   **Delete:** After emptying, go to Tab **Properties** → Click **Delete** → Type the Bucket name to confirm.

#### **5\. Delete Networking and DNS**

1.  **Delete NAT Gateway (CRITICAL COST STEP):**

    -   Go to **VPC** → **NAT Gateways**. Select Gateway → Click **Actions** → **Delete NAT Gateway**.

2.  **Delete Internet Gateway (IGW):**

    -   Go to **VPC** → **Internet Gateways**. Select IGW → Click **Actions** → **Detach from VPC**.

    -   Select IGW → Click **Actions** → **Delete Internet Gateway**.

3.  **Delete VPC Components:**

    -   Delete **Security Groups** → Delete **Subnets** → Delete **VPC** itself.

4.  **Delete Route 53 Hosted Zone:**

    -   Go to **Route 53** → **Hosted Zones**. Delete all custom records → Click **Delete Hosted Zone**.

* * * * *

### **III. Final Verification**

-   **AWS Billing/Cost Explorer:** Immediately check to ensure your estimated hourly cost has dropped to **$0.00**.

-   **Dashboards:** Verify the EC2, ECS, and RDS dashboards in **ap-southeast-1** show zero running resources.