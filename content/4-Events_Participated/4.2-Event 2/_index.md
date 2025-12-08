+++
title = "Event 2"
weight = 2
chapter = false
pre = " <b> 4.2.  </b> "
+++

# AWS Cloud Mastery Series #3 – Security Pillar Workshop

This workshop focused on helping participants understand how to design and operate secure AWS systems based on the **Security Pillar of the AWS Well-Architected Framework**, combining theory with practical demos and real-world scenarios.

---

## What I Learned

### Security Foundation

The session started with an overview of the **Security Pillar** and its role in cloud architecture. Key security principles such as **Least Privilege, Zero Trust, and Defense in Depth** were explained in a practical context.  
The speaker also clarified the **AWS Shared Responsibility Model**, highlighting common misunderstandings and real security risks seen in cloud environments in Vietnam.

---

### Identity & Access Management (IAM)

This part covered modern IAM design and best practices, including:
- How to use **IAM Roles instead of long-term credentials**
- Managing access with **IAM Users, Roles, and Policies**
- Using **IAM Identity Center** for SSO and permission sets
- Applying **Service Control Policies (SCP)** and permission boundaries in multi-account setups
- Enforcing security with **MFA, credential rotation, and Access Analyzer**

A short demo demonstrated how to **validate IAM policies and simulate access**, showing how misconfigurations can easily lead to over-permission.

---

### Detection & Monitoring

The workshop then moved on to detection and monitoring, where I learned how AWS provides visibility into security events:
- Using **CloudTrail**, **GuardDuty**, and **Security Hub** for continuous detection
- Implementing logging at multiple layers (VPC Flow Logs, ALB logs, S3 logs)
- Automating alerts and responses with **EventBridge**
- Applying the concept of **Detection-as-Code** to make detection consistent and repeatable

---

### Infrastructure Protection

This section focused on securing the network and compute layer:
- Designing VPCs with proper **segmentation and public/private placement**
- Understanding the difference between **Security Groups and Network ACLs**
- Protecting workloads with **WAF, Shield, and Network Firewall**
- Basic security practices for **EC2 and container workloads (ECS/EKS)**

The examples helped connect architectural decisions to real security outcomes.

---

### Data Protection

Data security was covered through practical encryption and access strategies:
- Managing encryption keys with **AWS KMS** (policies, grants, rotation)
- Encrypting data **at rest and in transit** across services like S3, EBS, RDS, and DynamoDB
- Storing and rotating secrets using **Secrets Manager** and **Parameter Store**
- Applying **data classification and guardrails** to reduce the risk of data leakage

---

### Incident Response

The final technical session focused on **incident response**:
- AWS-recommended IR lifecycle
- Playbooks for real-world cases such as:
  - Compromised IAM credentials
  - Public S3 bucket exposure
  - Malware detected on EC2
- Techniques for isolation, snapshotting, and evidence collection
- Automating responses with **Lambda** and **Step Functions**

This section highlighted the importance of preparing response workflows **before** incidents occur.

---

## Key Takeaways

- I gained a clear understanding of how the **five Security Pillars** work together in AWS.
- Security is not just about tools, but about **design, automation, and continuous monitoring**.
- Proper IAM design is critical and often the weakest point in cloud environments.
- Incident response should be automated and tested, not handled manually under pressure.

> Overall, the workshop provided practical insights into securing AWS environments and helped strengthen my confidence in designing and operating cloud systems with security in mind.
