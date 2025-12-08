+++
title = "Backend Deploy"
weight = 2
chapter = false
pre = " <b> 5.3.2. </b> "
alwaysopen = true
+++

### Architecture Model

This model illustrates the core services used for high-availability, low-latency content delivery and security:

1.  **Domain & SSL:** Route 53 (DNS) is used for domain management, and ACM (SSL Certificate) secures the traffic.

2.  **CDN:** CloudFront acts as the Global Edge Network for content delivery.

3.  **Storage (Primary):** S3 in Singapore (`ap-southeast-1`) holds the main static assets.

4.  **Storage (Failover):** S3 in N. Virginia (`us-east-1`) serves as the secondary storage location.

5.  **Replication:** Automated replication copies files from Singapore to Virginia for redundancy.

6.  **Security:** OAC (Origin Access Control) is implemented to keep the S3 buckets private and ensure traffic only flows via CloudFront.

### **Table of Contents**

1.  [Network & Security Preparation (VPC, SG)]({{% relref "5.3.2.1-Network & Security Preparation (VPC, SG)" %}})

2.  [Infrastructure & ALB Setup (RDS, Redis, Cloud Map, ALB Routing)]({{% relref "5.3.2.2-Infrastructure & ALB Setup (RDS, Redis, Cloud Map, ALB Routing)" %}})

3.  [Code Update & Image Build (Create new Docker Image)]({{% relref "5.3.2.3-Code Update & Image Build (Create new Docker Image)" %}})

4.  [Task Definitions Creation (Configure settings, FIX environment variables)]({{% relref "5.3.2.4-Task Definitions Creation (Configure settings, FIX environment variables)" %}})

5.  [Services Deployment ]({{% relref "5.3.2.5-Services Deployment" %}})

6.  [Completion & Verification (Route 53, Google Console, Final Test)]({{% relref "5.3.2.6-Completion & Verification (Route 53, Google Console, Final Test)" %}})
