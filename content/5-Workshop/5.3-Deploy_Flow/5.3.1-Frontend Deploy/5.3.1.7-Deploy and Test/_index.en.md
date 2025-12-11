+++
title = "Deploy and Test"
weight = 7
chapter = false
pre = " <b> 5.3.1.7. </b> "
+++

### PHASE 6: DEPLOY & TESTING

#### Step 6.1: Build & Deploy

On your local computer (in the React project folder):

```bash
# 1. Build to the production folder
npm run build

# 2. Upload to the MAIN bucket (Singapore)
# Note: Just upload to Sing, AWS will automatically copy to US
aws s3 sync build/ s3://sgutodolist-frontend-sg --delete

# 3. Clear the CloudFront cache so users can see the new code immediately
aws cloudfront create-invalidation --distribution-id <ID_CUA_BAN> --paths "/*"

```
#### Step 6.2: Test

1. Go to `https://sgutodolist.com`.

2. Try reloading the page (F5) at the sub-links (e.g. `/tasks`) to see if there is a 404 error.

3. Test Replication: Go to S3 Console bucket Virginia to see if the file has appeared automatically (usually takes 15s - 1 minute).

<!-- {{< figurecaption src="/images/fe6.2_2.jpg" caption="">}} -->
![](/AWS-Workshop/images/fe6.2_2.jpg)


* * * * *
<div style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px;">
<a href="{{% relref "5-Workshop/5.3-Deploy_Flow/5.3.1-Frontend Deploy/5.3.1.6-DNS Record" %}}" style="text-decoration: none; font-weight: bold;">
⬅ STEP 6: DNS Record
</a>

</div>