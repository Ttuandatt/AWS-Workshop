+++
title = "Completion & Verification (Route 53, Google Console, Final Test)"
weight = 6
chapter = false
pre = " <b> 5.3.2.6. </b> "
alwaysopen = true
+++

This final phase connects the deployed backend to the public domain and verifies end-to-end functionality.

### DNS Configuration

**Step 1: Create Route 53 A Record**

1.  Navigate to **Route 53** → **Hosted zones**
2.  Select `sgutodolist.com` hosted zone
3.  Click **Create record**
4.  Configure record:
    -   **Record name**: `api`
    -   **Record type**: A
    -   **Alias**: Enable
    -   **Route traffic to**: Alias to Application Load Balancer
    -   **Region**: Asia Pacific (Singapore)
    -   **Load balancer**: Select `sgu-alb`
5.  Click **Create records**

**DNS Propagation:** Wait 2-5 minutes for DNS changes to propagate.

**Verification:**

bash

```
nslookup sgutodolist.com
# Should return ALB's IP addresses
```

* * * * *

### Google OAuth Configuration Update

**Step 1: Update Authorized Redirect URIs**

1.  Access **Google Cloud Console** (console.cloud.google.com)
2.  Navigate to **APIs & Services** → **Credentials**
3.  Select your OAuth 2.0 Client ID
4.  Under **Authorized redirect URIs**, add:

```
   https://sgutodolist.com/api/auth/login/oauth2/code/google
```

1.  Click **Save**

**Note:** Keep existing localhost URIs for local development.

* * * * *

### System Health Verification

Perform the following tests to verify deployment success:

#### Test 1: API Gateway Health Check

bash

```
curl https://sgutodolist.com/actuator/health
```

**Expected Response:**

json

```
{
  "status": "UP"
}
```

#### Test 2: Individual Service Health Checks

bash

```
# Auth Service
curl https://sgutodolist.com/api/auth/actuator/health

# User Service
curl https://sgutodolist.com/api/user/actuator/health

# Taskflow Service
curl https://sgutodolist.com/api/taskflow/actuator/health

# Notification Service
curl https://sgutodolist.com/api/notification/actuator/health
```

All should return `{"status":"UP"}`.

#### Test 3: Service Discovery Verification

From the Bastion Host, verify internal DNS resolution:

bash

```
# SSH to Bastion
ssh -i sgutodolist-key.pem ec2-user@[BASTION-IP]

# Test DNS resolution
nslookup auth.sgu.local
nslookup user.sgu.local
nslookup taskflow.sgu.local
nslookup notification.sgu.local
nslookup ai-model.sgu.local
nslookup kafka.sgu.local
```

All should resolve to internal ECS task IP addresses.

#### Test 4: Database Connectivity

Verify services can connect to RDS:

1.  Check CloudWatch Logs for any service
2.  Look for successful database connection messages
3.  Verify no connection errors in startup logs

#### Test 5: Redis Connectivity

bash

```
# From Bastion Host
redis-cli -h [REDIS-ENDPOINT] ping
# Expected response: PONG
```

#### Test 6: End-to-End Authentication Flow

1.  Access frontend at `https://sgutodolist.com`
2.  Click "Sign in with Google"
3.  Complete OAuth flow
4.  Verify successful login and token issuance
5.  Verify user profile loads correctly

* * * * *

### Performance Baseline

Record initial performance metrics:

**Response Time Benchmarks:**

bash

```
# API Gateway response time
time curl -o /dev/null -s https://sgutodolist.com/actuator/health

# Auth service response time
time curl -o /dev/null -s https://sgutodolist.com/api/auth/actuator/health
```

**CloudWatch Metrics to Monitor:**

-   ECS Task CPU Utilization
-   ECS Task Memory Utilization
-   ALB Target Response Time
-   ALB Request Count
-   RDS CPU Utilization
-   Redis CPU Utilization

* * * * *

### Post-Deployment Security Checklist

-   [ ]  All sensitive environment variables secured (not in version control)
-   [ ]  Database password meets complexity requirements
-   [ ]  Security groups follow least privilege principle
-   [ ]  SSL/TLS certificates valid and auto-renewal enabled
-   [ ]  Bastion Host accessible only from authorized IPs
-   [ ]  CloudWatch Logs retention configured
-   [ ]  AWS Budget alerts active

* * * * *

### Final Deployment Checklist

**Infrastructure:**

-   [ ]  VPC and subnets operational
-   [ ]  All 4 security groups correctly configured
-   [ ]  RDS database accessible and initialized
-   [ ]  Redis cache operational
-   [ ]  Kafka service running
-   [ ]  ALB active with healthy targets

**Application:**

-   [ ]  All 6 services deployed and running
-   [ ]  Service Discovery functional
-   [ ]  ALB routing rules working correctly
-   [ ]  Health checks passing
-   [ ]  CloudWatch Logs collecting data

**Integration:**

-   [ ]  DNS record pointing to ALB
-   [ ]  SSL certificate valid
-   [ ]  Google OAuth configured
-   [ ]  Frontend can communicate with backend
-   [ ]  Authentication flow working

**Monitoring:**

-   [ ]  CloudWatch dashboards created
-   [ ]  Budget alerts configured
-   [ ]  Performance baseline recorded

* * * * *

### Known Limitations and Future Improvements

**Current Architecture Constraints:**

1.  **Single-AZ Database**: RDS is deployed in a single availability zone for cost optimization
2.  **Single-Node Redis**: No automatic failover for cache layer
3.  **Single-Node Kafka**: Not production-grade for high-throughput scenarios
4.  **Public Subnet ECS Tasks**: Security trade-off for cost savings

**Recommended Production Enhancements:**

1.  Enable RDS Multi-AZ deployment
2.  Implement Redis Cluster Mode with multiple replicas
3.  Deploy Kafka with multiple brokers across AZs
4.  Add NAT Gateway and move ECS tasks to private subnets
5.  Implement AWS WAF on ALB for DDoS protection
6.  Enable ECS Service Auto Scaling
7.  Implement CI/CD pipeline for automated deployments

* * * * *

Troubleshooting Guide
---------------------

### Quick Diagnosis Commands

bash

```
# Check ECS service status
aws ecs describe-services --cluster [CLUSTER-NAME] --services [SERVICE-NAME] --region ap-southeast-1

# Check task status
aws ecs describe-tasks --cluster [CLUSTER-NAME] --tasks [TASK-ARN] --region ap-southeast-1

# View recent logs
aws logs tail /ecs/[SERVICE-NAME] --follow --region ap-southeast-1

# Check target health
aws elbv2 describe-target-health --target-group-arn [TG-ARN] --region ap-southeast-1
```

### Emergency Rollback Procedure

If deployment fails:

1.  **Identify failing service**:

bash

```
   aws ecs list-services --cluster [CLUSTER-NAME] --region ap-southeast-1
```

1.  **Update service to previous task definition revision**:

bash

```
   aws ecs update-service\
     --cluster [CLUSTER-NAME]\
     --service [SERVICE-NAME]\
     --task-definition [TASK-DEF-FAMILY]:[PREVIOUS-REVISION]\
     --region ap-southeast-1
```

1.  **Force new deployment**:

bash

```
   aws ecs update-service\
     --cluster [CLUSTER-NAME]\
     --service [SERVICE-NAME]\
     --force-new-deployment\
     --region ap-southeast-1
```

* * * * *

Success Criteria
----------------

Deployment is considered successful when:

1.  ✅ All 6 ECS services show status: RUNNING
2.  ✅ All 5 target groups show: Healthy
3.  ✅ `https://sgutodolist.com/actuator/health` returns HTTP 200
4.  ✅ Frontend at `https://sgutodolist.com` can authenticate via Google OAuth
5.  ✅ CloudWatch Logs show no critical errors
6.  ✅ All services accessible via internal DNS (*.sgu.local)

* * * * *

<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.2-Backend Deploy/5.3.2.5-Services Deployment" %}}" style="text-decoration: none; font-weight: bold;">
⬅ BƯỚC 5: Code Update & Image Build
</a>
<a href="" style="text-decoration: none; font-weight: bold;">
</a>
</div>