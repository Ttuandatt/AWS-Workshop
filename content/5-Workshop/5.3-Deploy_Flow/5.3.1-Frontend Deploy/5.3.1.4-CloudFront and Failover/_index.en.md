+++
title = "CloudFront and Failover"
weight = 4
chapter = false
pre = " <b> 5.3.1.4. </b> "
+++

### STAGE 3: CONFIGURATION CLOUDFRONT (CDN & FAILOVER)

### STEP 3.1: Create Distribution

#### Step 1: Get started

- **Distribution name:** `sgutodolist-frontend-cloudfront`

- **Distribution type:** Keep **Single website or app**.

- **Domain:**

- In the **Route 53 managed domain** box, enter: `sgutodolist.com`.

- (If there is an Alternate domain names box, enter `www.sgutodolist.com` if the interface allows, otherwise we will add it later).

- Click **Next**.

<!-- {{< figurecaption src="/images/fe3.1_1.jpg" caption="">}} -->
![](/AWS-Workshop/images/fe3.1_1.jpg)



#### Step 2: Specify origin

- **Origin type:** Select **Amazon S3**.

- **Origin:** 

- Click on the search box, select bucket Singapore: `sgutodolist-frontend-sg...`.


- **Allow private S3 bucket access to CloudFront:** 

- Select: **Allow private S3 bucket access to CloudFront - Recommended**.

- **Cache settings:** Leave "Use recommended cache settings..." unchanged.

- Click **Next**.

<!-- {{< figurecaption src="/images/fe3.1_2.jpg" caption="">}} -->
![](/AWS-Workshop/images/fe3.1_2.jpg)


#### Step 3: Enable security

- **Web Application Firewall (WAF):**

- Check the box on the right: **Do not enable security protections** (to save costs).

- Click **Next**.

<!-- {{< figurecaption src="/images/fe3.1_3.jpg" caption="">}} -->
![](/AWS-Workshop/images/fe3.1_3.jpg)


#### Step 4: Get TLS certificate

- **TLS certificate:**

- Select the generated ACM certificate: `sgutodolist.com (...)`.

- Click **Next**.

<!-- {{< figurecaption src="/images/fe3.1_4.jpg" caption="">}} -->
![](/AWS-Workshop/images/fe3.1_4.jpg)


#### Step 5: Review and create

- Scroll down to the bottom and click the orange button **Create distribution**.

<!-- {{< figurecaption src="/images/fe3.1_5.jpg" caption="">}} -->
![](/AWS-Workshop/images/fe3.1_5.jpg)


* * * * *

### STEP 3.1 (Additional): Post-creation configuration (Required)

**Origin access control:** This step is done after successfully creating a Distribution, go to the Origin tab of the newly created Distribution, click on origin and click edit

- Click **Create new OAC**.

- Name: `S3-OAC-HA`.

- Signing behavior: **Sign requests**.

- Click **Create**.

<!-- {{< figurecaption src="/images/fe3.1_6.jpg" caption="">}} -->
![](/AWS-Workshop/images/fe3.1_6.jpg)


Next:

**1\. Copy Policy (Important):**

- Click on the newly created Distribution

- Go to the Origin tab of the newly created Distribution

- Click on the origin in the Origins list and click the Edit button

<!-- {{< figurecaption src="/images/fe3.1_7.jpg" caption="">}} -->
![](/AWS-Workshop/images/fe3.1_7.jpg)


- Click the **Copy policy** button in the Origin access control.

<!-- {{< figurecaption src="/images/fe3.1_8.jpg" caption="">}} -->
![](/AWS-Workshop/images/fe3.1_8.jpg)


- Go to the S3 Console tab > Bucket Singapore > Permissions > Bucket Policy > Paste > Save.

<!-- {{< figurecaption src="/images/fe3.1_9.jpg" caption="">}} -->
![](/AWS-Workshop/images/fe3.1_9.jpg)


**2\. Add Default Root Object (Fix white screen error):**

- In the newly created Distribution details screen, select the **General** tab (first tab).

- Scroll down to the **Settings** section, click the **Edit** button (located to the right of the Settings section).

- Find the **Default root object** box.

- Enter: `index.html`.

- (By the way, check the **Alternate domain names (CNAMEs)** section: Make sure both `sgutodolist.com` and `www.sgutodolist.com` are present. If missing, add the item).

- Scroll down and click **Save changes**.

<!-- {{< figurecaption src="/images/fe3.1_10.jpg" caption="">}} -->
![](/AWS-Workshop/images/fe3.1_10.jpg)

#### Step 3.2: Add Secondary Origin (Virginia)

1. Go to the newly created Distribution > **Origins** tab.

2. Click **Create origin**.

- **Origin domain:** Select the Virginia bucket (`sgutodolist-frontend-us.s3...`).

- **Origin access:** Reselect the `S3-OAC-HA` created earlier.

- **Name:** `Failover-US`.

3. Click **Create origin**.

<!-- {{< figurecaption src="/images/fe3.2_1.jpg" caption="">}} -->
![](/AWS-Workshop/images/fe3.2_1.jp)

<!-- {{< figurecaption src="/images/fe3.2_2.jpg" caption="">}} -->
![](/AWS-Workshop/images/fe3.2_2.jp)


#### Step 3.3: Create Origin Group (Enable High Availability)

1. Still on the Origins tab > Click **Create origin group**.

2. **Name:** `HighAvailability-Group`.

3. **Origins:**

- Add `Primary-SG` (Up - Priority 1).

- Add `Failover-US` (Down - Priority 2).

4. **Failover criteria:** Select: **500, 502, 503, 504**.

5. Click **Create origin group**.

<!-- {{< figurecaption src="/images/fe3.3_1.jpg" caption="">}} -->
![](/AWS-Workshop/images/fe3.3_1.jpg)


We will have 2 origins and 1 origin group:
<!-- {{< figurecaption src="/images/fe3.3_2.jpg" caption="">}} -->
![](/AWS-Workshop/images/fe3.3_2.jpg)



#### Step 3.4: Update Behavior

1. **Behaviors** Tab > Select `Default (*)` > **Edit**.

2. **Origin and origin groups:** Change from `Primary-SG` to **`HighAvailability-Group`**.

3. Click **Save changes**.

<!-- {{< figurecaption src="/images/fe3.4_1.jpg" caption="">}} -->
![](/AWS-Workshop/images/fe3.4_1.jpg)


#### Step 3.5: Configure SPA Routing (Handle 404 React errors)

1. Tab **Error pages** > **Create custom error response**.

2. **Rule 1 (For OAC):**

- HTTP error code: **403**.

- Customize error response: **Yes**.

- Response page path: `/index.html`.

- HTTP response code: **200**.

<!-- {{< figurecaption src="/images/fe3.5_1.jpg" caption="">}} -->
![](/AWS-Workshop/images/fe3.5_1.jpg)


3. **Rule 2 (For React Router):**

- Create another similar one for the code **404**. (Path is still `/index.html`, code 200).

<!-- {{< figurecaption src="/images/fe3.5_2.jpg" caption="">}} -->
![](/AWS-Workshop/images/fe3.5_2.jpg)



* * * * *
<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.3-Route 53 and ACM" %}}" style="text-decoration: none; font-weight: bold;">
⬅ STEP 3: Route 53 and ACM
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.5-S3 Policy" %}}" style="text-decoration: none; font-weight: bold;">
STEP 5: S3 Policy ➡
</a>
</div>