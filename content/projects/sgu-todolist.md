+++
title = "SaaS Task Management Web Application"
description = "A Todoist-like SaaS platform built with Spring Boot microservices, Aurora MySQL, Redis, and Kafka on AWS. Features JWT/OAuth2 authentication, real-time notifications, AI-powered task labeling, and CI/CD with GitHub Actions."
weight = 1
chapter = false
layout = "todolist-proposal"
tags = ["Spring Boot", "AWS", "Microservices", "Kafka", "JWT/OAuth2", "Docker", "CI/CD"]
image = "/images/todolist-architecture-6.jpg"
+++

## Overview

**AWS TodoList** is a production-grade task management platform delivering a **Todoist-like experience** — built with a Microservices architecture on AWS Cloud. The system supports real-time collaboration, project-based task management with role-based access control, personal inboxes, and AI-powered task labeling.

Developed during a **6-month internship at Amazon Web Services Vietnam** in a **4-member team**, the platform decomposes business logic into **6 independent microservices** communicating through Apache Kafka and deployed on AWS ECS Fargate.

🔗 **GitHub**: [aws-todo-list-project](https://github.com/gnuh24/aws-todo-list-project)
🎥 **Video Demo**: [YouTube](https://youtu.be/gOVHkb54aeM)
📂 **Workshop Documentation**: [AWS Workshop](https://ttuandatt.github.io/AWS-Workshop/5-workshop/)

---

## System Architecture

![Architecture Diagram](/images/todolist-architecture-6.jpg)

The platform follows a **Microservices Architecture** with Spring Cloud Gateway as the single entry point. All inter-service communication happens through **Apache Kafka** for async events and direct HTTP calls via Docker internal networking.

### Request Flow

```
Client → API Gateway (JWT Validation)
              ↓
    ┌─────────┴──────────┐
    │  Extract JWT Claims │
    │  Inject X-User-Id   │
    │  Inject X-User-Email│
    │  Inject X-User-Role │
    └─────────┬──────────┘
              ↓
    Route to Target Service
    (Auth / User / Taskflow / Notification)
              ↓
    Service processes request
              ↓
    Kafka Event Published
    (Notification / Cache Invalidation)
```

---

## Backend Microservices

### 🌐 API Gateway (Port 8080)

The single entry point for all client requests, built with **Spring Cloud Gateway**.

**Key Implementation:**
- **GlobalFilter** (`JwtGatewayFilter`) — Intercepts every request, validates JWT tokens, and injects user context headers (`X-User-Id`, `X-User-Email`, `X-User-Role`) before routing to downstream services
- **Public Path Bypass** — Whitelisted endpoints (login, register, OAuth2 callback) skip JWT validation
- **Token Type Validation** — Only `access` tokens are accepted; `refresh` tokens are explicitly rejected at gateway level
- **Rate Limiting** — Redis-backed rate limiter to prevent API abuse
- **Service Routing** — Routes to auth-service (9999), user-service (8081), taskflow-service (8082), notification-service (9998), and AI model service (9997)

---

### 🔐 Auth Service (Port 9999) — *My Primary Contribution*

The authentication and authorization backbone of the entire platform. This was my core development responsibility.

**Authentication Endpoints:**

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/v1/login` | POST | Traditional email/password login |
| `/v1/staff-login` | POST | Staff-specific login with elevated permissions |
| `/v1/register` | POST | User registration with OTP email verification |
| `/v1/active-account` | POST | Account activation via OTP code |
| `/v1/refresh-token` | POST | Access token renewal using refresh token (Cookie-based) |
| `/v1/check-email` | GET | Email existence validation |
| `/v1/send-reset-password-otp/{email}` | POST | OTP-based password reset initiation |
| `/v1/reset-password/{username}` | PATCH | Password reset with OTP verification |
| `/v1/update-password` | PATCH | Authenticated password change |
| `/v1/update-email` | PATCH | Email update with OTP verification |
| `/v1/send-update-email-otp/{newEmail}` | POST | OTP for email change |

**Security Implementation:**
- **JWT Token System** — Dual-token architecture with Access Token + Refresh Token, each carrying `typ`, `role`, and `accountId` claims. Token signed with HMAC-SHA256 using Base64-encoded secret key
- **OAuth2 Social Login** — Google OAuth2 integration with custom `OAuth2LoginSuccessHandler` that auto-creates accounts and redirects with token data via URL parameters
- **Password Security** — BCrypt encoding via Spring Security's `PasswordEncoder`
- **OTP Verification** — Time-limited OTP codes (3 minutes) sent via email for account activation, password reset, and email updates
- **Stateless Sessions** — `SessionCreationPolicy.STATELESS` ensuring no server-side session storage
- **Redis Integration** — Token blacklisting for secure logout, OTP storage with TTL, session caching via `AccountRedisDTO`
- **Kafka Events** — Publishes registration and authentication events to Notification Service

**Exception Handling (10+ Custom Exceptions):**
- `AccessTokenBlacklistedException`, `AccessTokenExpiredException`, `InvalidJWTSignatureException`
- `RefreshTokenBlacklistedException`, `RefreshTokenExpiredException`, `RefreshTokenNotFound`
- `InvalidTokenTypeException`, `TokenExpiredException`, `HmacVerificationException`
- `StepUpAuthenticationException`, `OtpNotFoundException`

**Code Quality:**
- AOP-based request logging (`RequestLoggingFilter`) for debugging and monitoring
- Swagger/OpenAPI documentation with `@Operation` and `@Parameter` annotations
- MapStruct-based DTO mapping (`AuthMapper`)
- Clean layered architecture: Controller → Service → Repository

---

### 📋 Taskflow Service (Port 8082)

The core business logic engine handling all task and project management operations.

**Features:**
- **Project Management** — Create, update, archive, delete projects with multi-user collaboration
- **Role-Based Access Control (RBAC)** — 4 permission levels per project:
  - **Owner**: Full control, cannot be removed
  - **Admin**: Manage members, tasks, and sections (except Owner)
  - **Member**: Create and manage tasks
  - **Viewer**: Read-only access
- **Section Management** — Organize tasks into sections with drag-and-drop ordering
- **Task Management** — Full CRUD with status tracking (To-do, In-progress, Done, Blocked), priority levels (High, Medium, Low), assignees, and deadlines
- **Subtasks** — Nested subtask support with independent status and priority
- **Comments & Attachments** — Comment threads on tasks with file/image attachments
- **Personal Inbox** — Private task space independent of projects, with ability to convert inbox tasks to project tasks
- **Advanced Filtering** — JPA Specifications for dynamic filtering, sorting, and pagination
- **Pinning** — Pin/unpin important tasks for quick access

---

### 👤 User Service (Port 8081)

User profile management, separated from Auth to follow Single Responsibility Principle.

**Features:**
- User profile CRUD (display name, avatar upload, personal info)
- Profile data caching with Redis for fast reads
- Event-driven profile sync via Kafka

---

### 🔔 Notification Service (Port 9998)

Real-time notification engine powered by Kafka consumers and Redis Pub/Sub.

**Notification Types:**
- Project membership changes (added, role updated, project deleted)
- Task assignments (new task assigned, task updated by others)
- Task lifecycle events (completed, approaching deadline, overdue)
- Comment notifications (new comments on assigned/created tasks)
- Email notifications via SMTP

**Implementation:**
- Kafka Consumer for async event processing from all services
- In-app notification storage and delivery
- Redis Pub/Sub for real-time push to connected clients

---

### 🤖 AI Model Service (Port 9997)

Machine learning service for intelligent task management, written in **Python Flask**.

**Features:**
- Task label classification using a trained ML model (`model_label.joblib`)
- Smart tag suggestions based on task content
- REST API consumed by Taskflow Service for auto-labeling new tasks

---

## Infrastructure & DevOps

### Docker Compose Stack (9 Services)

```
┌─────────────────────────────────────────────────┐
│              Docker Compose Network             │
│                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────────┐   │
│  │  MySQL   │  │  Redis   │  │  Kafka 3.5   │   │
│  │   8.0    │  │   7.2    │  │ + Zookeeper  │   │
│  │ :3306    │  │ :6379    │  │ :29092/:9092 │   │
│  └──────────┘  └──────────┘  └──────────────┘   │
│                                                 │
│  ┌──────────────────────────────────────────┐   │
│  │        API Gateway (:8080)               │   │
│  │   → Auth (:9999) → User (:8081)          │   │
│  │   → Taskflow (:8082) → Notif (:9998)     │   │
│  │   → AI Model (:9997)                     │   │
│  └──────────────────────────────────────────┘   │
│                                                 │
│  ┌──────────┐                                   │
│  │ AKHQ UI  │  Kafka monitoring dashboard       │
│  │ :8083    │                                   │
│  └──────────┘                                   │
└─────────────────────────────────────────────────┘
```

### AWS Cloud Deployment

- **Compute**: ECS Fargate (serverless containers, auto-scaling)
- **Database**: RDS MySQL / Aurora (managed relational database)
- **Cache**: ElastiCache Redis (session caching, rate limiting, pub/sub)
- **Storage**: S3 (file uploads, static assets)
- **Load Balancer**: Application Load Balancer → API Gateway
- **Container Registry**: ECR (Docker image storage)
- **DNS & SSL**: Route 53 + ACM Certificate
- **Monitoring**: CloudWatch Logs, Metrics
- **CI/CD**: GitHub Actions → ECR → ECS (automated deployment, reduced manual release time by 50%)

---

## Technology Stack

### Backend
- **Language**: Java 21
- **Framework**: Spring Boot 3.4, Spring Security 6
- **ORM**: Hibernate / JPA + JPA Specifications
- **Gateway**: Spring Cloud Gateway (reactive, WebFlux)
- **Cache**: Spring Data Redis
- **Messaging**: Spring Kafka (producer/consumer)
- **Auth**: JWT (HMAC-SHA256) + OAuth2 (Google)
- **API Docs**: SpringDoc OpenAPI / Swagger
- **Mapping**: MapStruct (DTO ↔ Entity)
- **Build**: Maven, Lombok

### Frontend
- **Framework**: React + Vite
- **State**: Redux Toolkit
- **Styling**: TailwindCSS
- **HTTP**: Axios (dual client: user + admin)
- **Features**: Pagination, sort, filter, file upload

### Data & Messaging
- **RDBMS**: MySQL 8.0 / Aurora MySQL
- **Cache**: Redis 7.2 (caching, pub/sub, rate limiting, token blacklist)
- **Streaming**: Apache Kafka 3.5 (Bitnami) + Zookeeper 3.9
- **Monitoring**: AKHQ (Kafka UI)

### DevOps & Cloud
- **Containerization**: Docker, Docker Compose
- **Cloud**: AWS (ECS Fargate, ECR, RDS, ElastiCache, S3, ALB, VPC, IAM, Route 53, Cloud Map, CloudWatch, ACM)
- **CI/CD**: GitHub Actions
- **AI/ML**: Python Flask, joblib (ML model serving)

---

## Key Technical Decisions

| Decision | Why |
|----------|-----|
| **Spring Cloud Gateway** instead of Netflix Zuul | Reactive, non-blocking, WebFlux-based for better throughput |
| **JWT with Redis Blacklist** instead of session-based auth | Stateless, scalable, supports token revocation for logout |
| **Kafka** instead of RabbitMQ | Higher throughput, event replay capability, better for microservices |
| **JPA Specifications** for filtering | Dynamic, type-safe query building without raw SQL |
| **Separate Auth & User Services** | Single Responsibility, independent scaling |
| **Cookie-based Refresh Token** | More secure than localStorage, prevents XSS token theft |
| **AI Model as separate Flask service** | Language flexibility (Python ML ecosystem), independent deployment |

---

## My Contributions

1. **Auth Service** — Designed and implemented the entire authentication system: JWT token generation/validation, OAuth2 Google login, OTP-based email verification, password management, Redis-backed session handling
2. **API Gateway Integration** — Integrated Auth Service with the gateway's JWT filter for seamless request routing with user context propagation
3. **Docker Containerization** — Containerized services for local development and cloud deployment
4. **CI/CD Pipeline** — Implemented GitHub Actions pipelines for automated build, push to ECR, and deploy to ECS
5. **AWS Architecture Design** — Designed multi-region architecture with cost optimization strategies
6. **Team Collaboration** — Worked with 3 teammates to decompose monolith into 5 independent microservices with clear API contracts

---

## Outcomes

- **6 microservices** deployed and serving production traffic
- **162+ features** across 7 functional modules
- **50% reduction** in manual deployment time through CI/CD automation
- **~$30/month** operating cost through infrastructure optimization
- Comprehensive **AWS Workshop documentation** with 35+ deployment steps
