+++
title = "S3 Policy"
weight = 5
chapter = false
pre = " <b> 5.3.1.5. </b> "
+++

### STAGE 4: S3 PERMITS (POLICY)

CloudFront needs a "permission" to pull files from your 2 closed buckets.

1. In CloudFront > **Origins** tab.

2. Select Origin Singapore > Edit > **Copy policy**.

3. Open a new tab > S3 Console > Bucket `sgutodolist-frontend-sg`.

4. **Permissions** tab > **Bucket Policy** > Edit > Paste > Save.

5. **Repeat with Bucket Virginia:**

- Go back to CloudFront > Select Origin Virginia > Edit > Copy policy.

- Go to S3 `sgutodolist-frontend-us` > Permissions > Bucket Policy > Paste > Save.

* * * * *
<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.4-ClouFront and Failover" %}}" style="text-decoration: none; font-weight: bold;">
⬅ STEP 4: ClouFront and Failover
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.6-DNS Record" %}}" style="text-decoration: none; font-weight: bold;">
STEP 6: DNS Record ➡
</a>
</div>