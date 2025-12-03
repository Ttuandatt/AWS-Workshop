+++
title = "Frontend Deploy"
weight = 1
chapter = false
pre = " <b> 5.3.1. </b> "
alwaysopen = true
+++

HƯỚNG DẪN DEPLOY FRONTEND MULTI-REGION
======================================================

**Project:** SGU Task Management Web Application

**Domain:** sgutodolist.com

**Frontend:** ReactJS

**Architecture:** Multi-Region (Singapore + Virginia) + CloudFront CDN

**Thời gian thực hiện:** ~60-90 phút

MỤC LỤC
----------
1.  [Prerequisites]({{% relref "5.3.1.1-Prerequisites" %}})
    
2.  [Setup Domain & Route 53]({{% relref "5.3.1.2-Setup Domain & Route 53" %}})
    
3.  [Prepare Buckets (Singapore + Virginia)]({{% relref "5.3.1.3-Prepare Buckets" %}})
    
4.  [Setup Cross-Region Replication (CRR)]({{% relref "5.3.1.4-Cross-Region Replication" %}})
    
5.  [Create Multi-Region Access Point (MRAP)]({{% relref "5.3.1.5-Multi-Region Access Point" %}})
    
6.  [Setup CloudFront Distribution]({{% relref "5.3.1.6-CloudFront Distribution" %}})
    
7.  [Request SSL Certificate]({{% relref "5.3.1.7-SSL Certificate" %}})
    
8.  [Configure DNS Records]({{% relref "5.3.1.8-DNS Records" %}})
    
9.  [Testing & Verification]({{% relref "5.3.1.9-Testing & Verification" %}})
    
10. [Maintenance & Updates]({{% relref "5.3.1.10-Maintenance & Updates" %}})