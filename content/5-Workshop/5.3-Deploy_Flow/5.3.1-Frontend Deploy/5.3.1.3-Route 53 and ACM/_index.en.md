+++
title = "Route 53 and ACM"
weight = 3
chapter = false
pre = " <b> 5.3.1.3.  </b> "
+++

## PHASE 2: DOMAIN & SECURITY PREPARATION (ROUTE 53 & ACM)

Before creating the CloudFront distribution, the domain name system (DNS) and SSL certificates must be properly prepared.

---

### Step 2.1: Create a Hosted Zone (If Not Exists)

1. Navigate to **Amazon Route 53** → **Hosted zones**.

2. If the domain `sgutodolist.com` does not exist in the list:

   - Click **Create hosted zone**.
   - **Domain name:** `sgutodolist.com`
   - **Type:** Public hosted zone
   - Click **Create**.

{{< figurecaption src="/images/fe2.1_1.jpg" caption="">}}

3. Update Name Servers at the domain provider  
   (This project uses a domain purchased from a third-party registrar.)

   - After opening the newly created Hosted Zone, you will see four Name Servers in the following format:

ns-1538.awsdns-00.co.uk.
ns-1374.awsdns-43.org.
ns-172.awsdns-21.com.
ns-547.awsdns-04.net.

yaml
Copy code

- Copy all four Name Servers and update them in the domain management panel of your domain registrar.

{{< figurecaption src="/images/fe2.1_2.jpg" caption="">}}

---

### Step 2.2: Request an SSL Certificate (ACM) – IMPORTANT

⚠️ **IMPORTANT NOTE:**  
SSL certificates used with **Amazon CloudFront** **MUST** be created in the **US East (N. Virginia)** region.

1. Switch the AWS Console region to **US East (N. Virginia)**.

2. Go to **AWS Certificate Manager (ACM)** → **Request certificate**.

3. Select **Request a public certificate** → **Next**.

4. **Domain names:**

- `sgutodolist.com`
- `*.sgutodolist.com`  
  (Wildcard domain for subdomains such as `www`.)

5. **Validation method:** DNS validation (Recommended).

6. Click **Request**.

7. In the Certificates list, click the newly created certificate  
(Status: **Pending validation**).

8. Under the **Domains** section, click **Create records in Route 53**.

9. Click **Create records**.

10. Wait a few minutes until the certificate status changes to **Issued** (Green).

{{< figurecaption src="/images/fe2.2_1.jpg" caption="">}}

{{< figurecaption src="/images/fe2.2_2.jpg" caption="">}}

{{< figurecaption src="/images/fe2.2_3.jpg" caption="">}}

---

<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.2-S3 and Replication" %}}" style="text-decoration: none; font-weight: bold;">
⬅ STEP 2: S3 and Replication
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.4-ClouFront and Failover" %}}" style="text-decoration: none; font-weight: bold;">
STEP 4: CloudFront and Failover ➡
</a>
</div>
