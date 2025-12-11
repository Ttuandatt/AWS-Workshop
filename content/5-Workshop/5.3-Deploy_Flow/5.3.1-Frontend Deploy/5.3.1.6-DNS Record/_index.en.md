+++
title = "DNS Record"
weight = 6
chapter = false
pre = " <b> 5.3.1.6. </b> "
+++

### PHASE 5: POINT OFFICIAL DOMAIN (DNS RECORD)

1. Go to **Route 53** > **Hosted zones** > `sgutodolist.com`.

2. **Create Record for Root domain:**

- Click **Create record**.

- Record name: (leave blank).

- Type: **A**.

- Alias: **Yes** (Swipe the button to the right).

- Route traffic to: **Alias ​​to CloudFront distribution**.

- Choose distribution: Select the CloudFront domain (e.g. `d123...cloudfront.net`).

- Click **Create records**.

<!-- {{< figurecaption src="/images/fe5_1.jpg" caption="">}} -->
![](/AWS-Workshop/images/fe5_1.jpg)


3. **Create Record for WWW:**

- Do the same, but fill in `www` for Record name.

<!-- {{< figurecaption src="/images/fe5_2.jpg" caption="">}} -->
![](/AWS-Workshop/images/fe5_2.jpg)


* * * * *
<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.5-S3 Policy" %}}" style="text-decoration: none; font-weight: bold;">
⬅ STEP 5: S3 Policy
</a>
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.7-Deploy and Test" %}}" style="text-decoration: none; font-weight: bold;">
STEP 7: Deploy and Test ➡
</a>
</div>