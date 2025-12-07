+++
title = "Prerequisite"
weight = 2
chapter = false
pre = " <b> 5.2.  </b> "
+++

### **Preamble: Prerequisites for Backend Deployment**

Before initiating the deployment steps outlined in the Table of Contents, the following foundational tools, configurations, and AWS service setups must be completed to ensure a smooth, secure, and cost-optimized deployment into the **ap-southeast-1 (Singapore)** region.

| **Step** | **Requirement** | **Purpose** |
| --- | --- | --- |
| 1 | **AWS Account & CLI** | Access and command-line management. |
| 2 | **Domain & SSL** | Securing traffic and global content delivery. |
| 3 | **Source Code & Artifacts** | Building the deployable microservice images. |
| 4 | **AWS Network Foundation** | Setting up the isolated, highly available VPC structure. |

* * * * *

### **I. Tools & Credentials**

1.  **AWS Account Setup:** A valid AWS Account is required, and initial **billing alerts (AWS Budgets)** must be configured to monitor Free Tier usage.

2.  **AWS CLI:** AWS Command Line Interface must be installed and configured with appropriate **IAM User credentials** granting necessary permissions (EC2, RDS, S3, CloudFront, Route 53, IAM, CloudWatch).

3.  **Docker:** Docker Engine must be installed locally to build the container images for the Spring Boot microservices.

4.  **Java/Maven:** Local environment must be set up to compile and package the Spring Boot application code.

### **II. Domain, SSL, and DNS Preparation**

1.  **Registered Domain:** A domain name must be registered (e.g., via Route 53 or an external registrar).

2.  **Route 53 Hosted Zone:** The domain must be managed within an **Amazon Route 53 Hosted Zone**.

3.  **ACM Certificate:** An **AWS Certificate Manager (ACM)** certificate must be provisioned for the domain (e.g., `*.taskmanager.com`) in **two regions**:

    -   **us-east-1 (N. Virginia):** Required for CloudFront (Global Service).

    -   **ap-southeast-1 (Singapore):** Required for the Application Load Balancer (ALB).

### **III. Network Foundation (VPC & Subnets)**

A VPC must be created in **ap-southeast-1** with the following structure for High Availability (HA) across at least two Availability Zones (AZs):

1.  **VPC:** One primary VPC (e.g., CIDR 10.0.0.0/16).

2.  **Public Subnets (Multi-AZ):** To host the **Application Load Balancer (ALB)** and **Internet Gateway (IGW)**.

3.  **Private Subnets (Multi-AZ):** To host the application instances (**EC2/ECS**), **RDS**, and **ElastiCache**.

4.  **NAT Gateway:** Deployed in at least one Public Subnet to allow resources in the Private Subnets (EC2 instances) to securely access the internet for updates and communication with AWS services (like S3/ECR).

5.  **Security Groups (SG):** Initial Security Groups must be defined following the principle of least privilege:

    -   **ALB SG:** Allows inbound traffic on **HTTP (80) and HTTPS (443)** from the internet.

    -   **EC2/ECS SG:** Allows inbound traffic *only* from the **ALB SG** on the application port (e.g., 8080).

    -   **RDS/ElastiCache SG:** Allows inbound traffic *only* from the **EC2/ECS SG** on their respective ports (e.g., MySQL 3306, Redis 6379).

### **IV. Data & Storage Pre-Configuration**

1.  **S3 Primary Bucket (ap-southeast-1):** An S3 bucket must be created in `ap-southeast-1` to hold static assets and application deployment artifacts (JARs/Docker Images).

2.  **S3 DR Bucket (us-east-1):** A secondary S3 bucket must be created in `us-east-1` (N. Virginia) to serve as the Disaster Recovery target.

3.  **Cross-Region Replication (CRR):** **S3 CRR** must be configured to automatically replicate objects from the primary bucket in `ap-southeast-1` to the DR bucket in `us-east-1`.

### **V. Compute & Service Preparation**

1.  **Docker Images:** The Spring Boot microservices must be built into production-ready Docker images.

2.  **ECR Repository (Optional):** If using ECS, a repository must be prepared in **ECR (Elastic Container Registry)** to store the Docker images.

3.  **IAM Roles:** Necessary **IAM Roles** must be created for:

    -   **EC2/ECS Tasks:** Grants permissions to access S3, CloudWatch, and the RDS database.

    -   **ALB:** Allows the load balancer to perform its function.