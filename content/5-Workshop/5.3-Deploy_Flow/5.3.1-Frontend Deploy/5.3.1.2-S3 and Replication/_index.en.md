+++
title = "S3 and Replication"
weight = 2
chapter = false
pre = " <b> 5.3.1.2. </b> "
+++

## PHASE 1: STORAGE PREPARATION (S3 & REPLICATION)

This phase focuses on creating a durable storage layer for frontend resources and setting up an automated cross-region synchronization mechanism.

---

### Step 1.1: Create the Primary Bucket (Singapore)

1. Open the **Amazon S3 Console** and select the **Asia Pacific (Singapore)** region.

2. Click **Create bucket**.

   - **Bucket name:** `sgutodolist-frontend-sg`
   - **Object Ownership:** ACLs disabled (Recommended)
   - **Block Public Access:** ✅ **Block all public access**  
     (The bucket must remain private because CloudFront OAC will be used.)
   - **Bucket Versioning:** ✅ **Enable**  
     (Required for cross-region replication.)
   - **Default encryption:** Server-side encryption with Amazon S3 managed keys (SSE-S3)

3. Click **Create bucket**.

![](/AWS-Workshop/images/fe1.1_1.jpg)

![](/AWS-Workshop/images/fe1.1_2.jpg)

![](/AWS-Workshop/images/fe1.1_3.jpg)


---

### Step 1.2: Create the Secondary Bucket (N. Virginia)

1. Switch the region to **US East (N. Virginia)**.

2. Click **Create bucket**.

   - **Bucket name:** `sgutodolist-frontend-us`
   - **Block Public Access:** ✅ **Block all public access**
   - **Bucket Versioning:** ✅ **Enable**

3. Click **Create bucket**.

After completing **Step 1.1** and **Step 1.2**, two S3 buckets should be available:

- Primary bucket: Singapore (`ap-southeast-1`)
- Secondary bucket: N. Virginia (`us-east-1`)

<!-- {{< figurecaption src="/images/fe1.1_1.2.jpg" caption="">}} -->

![](/AWS-Workshop/images/fe1.1_1.2.jpg)


---

### Step 1.3: Configure Replication (Automatic Sync)

1. Go back to the **Singapore bucket** (`sgutodolist-frontend-sg`).

2. Navigate to **Management** → **Replication rules** → Click **Create replication rule**.

   - **Rule name:** `SyncToUS`
   - **Status:** Enabled
   - **Source bucket:** Apply to all objects in the bucket
   - **Destination:** Choose a bucket in this account  
     → Select `sgutodolist-frontend-us`  
     (Ensure that the region `us-east-1` is selected so the bucket is visible.)
   - **IAM Role:** Select **Create new role**  
     (AWS will automatically generate the required permissions.)

3. Click **Save**.  
   When prompted with **“Replicate existing objects?”**, select **No**  
   (The bucket is currently empty.)

{{< figurecaption src="/images/fe1.3_1.jpg" caption="">}}
![](/AWS-Workshop/images/fe1.3_1.jpg)

{{< figurecaption src="/images/fe1.3_2.jpg" caption="">}}
![](/AWS-Workshop/images/fe1.3_2.jpg)

{{< figurecaption src="/images/fe1.3_3.jpg" caption="">}}
![](/AWS-Workshop/images/fe1.3_3.jpg)

{{< figurecaption src="/images/fe1.3_4.jpg" caption="">}}
![](/AWS-Workshop/images/fe1.3_4.jpg)

---

<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.1-Prerequisites" %}}" style="text-decoration: none; font-weight: bold;">
⬅ STEP 1: Prerequisites
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.3-Route 53 and ACM" %}}" style="text-decoration: none; font-weight: bold;">
STEP 3: Route 53 and ACM ➡
</a>
</div>
