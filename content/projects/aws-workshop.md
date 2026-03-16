+++
title = "AWS Workshop - Deployment Documentation"
description = "Comprehensive step-by-step workshop documenting 35+ deployment steps for the SaaS platform on AWS. Covers VPC, ECS Fargate, ALB, RDS, CloudFront, and more."
weight = 3
chapter = false
layout = "project-detail"
tags = ["AWS", "ECS", "ALB", "VPC", "CloudFront", "Documentation"]
image = "/images/architecture-solution.png"
+++

## Overview

A comprehensive **step-by-step workshop** documenting the full deployment process of the SGU TodoList platform on AWS. This workshop was created as the primary deliverable during my internship, serving as both documentation and a learning resource for future interns.

---

## Workshop Contents

### 1. Workshop Overview
Goals, learning outcomes, and architecture review — covering the decisions behind the single-region cost-optimized infrastructure.

### 2. Prerequisites
Detailed setup guide for:
- AWS CLI configuration
- Docker Desktop setup
- Required IAM permissions and roles
- Domain registration and SSL certificates

### 3. Deploy Flow (Step-by-Step)
The core of the workshop with 35+ detailed deployment steps:

**Network Setup:**
- VPC creation (10.0.0.0/16)
- Public & Private subnet configuration across 2 AZs
- Internet Gateway and route tables
- Security Groups for ALB, ECS, RDS, Redis

**Backend Deployment:**
- ECR repository creation and Docker image push
- ECS Cluster setup with Fargate launch type
- Task Definition configuration for each microservice
- Service creation with rolling update deployment
- ALB configuration with path-based routing
- Cloud Map service discovery namespace setup

**Data Layer:**
- RDS MySQL instance provisioning
- ElastiCache Redis cluster setup
- Kafka deployment on ECS

**Frontend Deployment:**
- S3 bucket creation for static hosting
- CloudFront distribution with custom domain
- Cross-Region Replication (CRR) for disaster recovery

### 4. Clean Up
Step-by-step instructions to tear down all resources and avoid unexpected charges.

### 5. Mystic Skills
Additional advanced topics and best practices discovered during the deployment.

---

## Technologies Documented

- **AWS Services**: VPC, ECS Fargate, ALB, RDS, ElastiCache, S3, CloudFront, Route 53, Cloud Map, ECR, IAM, ACM, CloudWatch
- **Tools**: AWS CLI, Docker, PowerShell
- **Architecture**: Microservices, event-driven, cost-optimized single-region HA

---

## Impact

- **35+ deployment steps** with screenshots and explanations
- Available in **English and Vietnamese** (bilingual documentation)
- Serves as a reusable template for future AWS deployments
