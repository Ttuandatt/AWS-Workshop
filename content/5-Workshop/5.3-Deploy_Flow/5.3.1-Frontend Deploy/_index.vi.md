+++
title = "Triển khai Frontend"
weight = 1
chapter = false
pre = " <b> 5.3.1. </b> "
alwaysopen = true
+++


### Mô hình kiến trúc

1.  **Domain:** Route 53 (DNS) + ACM (SSL Certificate).

2.  **CDN:** CloudFront (Global Edge Network).

3.  **Storage (Primary):** S3 Singapore (`ap-southeast-1`).

4.  **Storage (Failover):** S3 N. Virginia (`us-east-1`).

5.  **Replication:** Tự động copy code từ Sing -> Virginia.

6.  **Security:** OAC (Origin Access Control) - Private Bucket.

### MỤC LỤC

1.  [Điều kiện tiên quyết]({{% relref "5.3.1.1-Prerequisites" %}})
    
2.  [S3 và Replication]({{% relref "5.3.1.2-S3 and Replication" %}})
    
3.  [Route 53 và ACM]({{% relref "5.3.1.3-Route 53 and ACM" %}})
    
4.  [ClouFront và Failover]({{% relref "5.3.1.4-ClouFront and Failover" %}})
    
5.  [S3 Policy]({{% relref "5.3.1.5-S3 Policy" %}})
    
6.  [DNS Record]({{% relref "5.3.1.6-DNS Record" %}})
    
7.  [Deploy và Test]({{% relref "5.3.1.7-Deploy and Test" %}})