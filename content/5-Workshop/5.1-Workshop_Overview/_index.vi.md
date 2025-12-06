+++
title = "Workshop Overview"
weight = 1
chapter = false
pre = " <b> 5.1.  </b> "
+++

Giới Thiệu Dự Án
-----------------

**SGU TodoList** là một ứng dụng quản lý công việc (Task Management) được xây dựng theo kiến trúc **Microservices** trên nền tảng AWS Cloud. Dự án được thiết kế ban đầu với mục tiêu triển khai theo mô hình **Multi-Region SaaS** để đảm bảo high availability và disaster recovery. Tuy nhiên, do giới hạn về ngân sách và tài khoản AWS Free Tier, nhóm đã tối ưu hóa kiến trúc về **Single-Region Deployment** với cơ chế **Cross-Region Failover** cho frontend.