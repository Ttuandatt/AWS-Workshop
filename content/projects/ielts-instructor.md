+++
title = "IELTS Helper — AI-Powered IELTS Preparation Platform"
description = "A full-stack web application for IELTS Reading & Writing practice with AI-powered essay scoring, real-time feedback, classroom management, and role-based dashboards. Built with Next.js 16, NestJS 10, PostgreSQL, BullMQ, and OpenAI/Gemini."
weight = 2
chapter = false
layout = "ielts-proposal"
tags = ["Next.js", "NestJS", "PostgreSQL", "AI/LLM", "BullMQ", "Redis", "Prisma", "TypeScript"]
+++

## Overview

**IELTS Helper** is a comprehensive web platform designed to solve real pain points in IELTS preparation:

- **Writing feedback latency**: Learners typically wait 1–3 days for teacher feedback — causing demotivation and slow improvement
- **Inconsistent scoring**: Different teachers score the same essay differently
- **Fragmented content**: IELTS materials are scattered across dozens of websites
- **No instant explanations**: Learners get a score but don't understand *why*

The platform delivers **auto-graded Reading with instant explanations** and **AI-powered Writing scoring** (4-criteria IELTS rubric, < 5 minutes turnaround), backed by a centralized content management system with role-based access for Learners, Instructors, and Admins.

**Status**: Actively developed (personal project)
**GitHub**: Private repository (available upon request)

---

## System Architecture

```
┌───────────────────────────────────────────────────────┐
│                    Client (Browser)                    │
│               Next.js 16 (App Router)                 │
│         TailwindCSS 4 · TanStack Query v5             │
│           i18n (vi/en) · Dark/Light Theme             │
└─────────────────────┬─────────────────────────────────┘
                      │ HTTPS (REST + SSE)
                      ▼
┌───────────────────────────────────────────────────────┐
│                  NestJS 10 Backend                     │
│               TypeScript · Prisma 6 ORM               │
│                                                       │
│  ┌──────────┐ ┌──────────┐ ┌────────────┐            │
│  │   Auth   │ │ Reading  │ │  Writing   │            │
│  │  Module  │ │  Module  │ │  Module    │            │
│  │ JWT+RBAC │ │ Auto-    │ │ BullMQ     │            │
│  │          │ │ Grading  │ │ Queue      │            │
│  └──────────┘ └──────────┘ └─────┬──────┘            │
│                                  │                    │
│  ┌──────────┐ ┌──────────┐ ┌────▼──────┐            │
│  │Classroom │ │  Admin   │ │ Scoring   │            │
│  │  Module  │ │  Module  │ │ Consumer  │            │
│  │ Topics & │ │ CRUD +   │ │ LLM Call  │            │
│  │ Lessons  │ │ Versions │ │ + Parse   │            │
│  └──────────┘ └──────────┘ └─────┬──────┘            │
│                                  │                    │
└──────────┬───────────────────────┼────────────────────┘
           │                       │
     ┌─────▼─────┐          ┌─────▼──────────────┐
     │PostgreSQL │          │  OpenAI GPT-4o     │
     │   15      │          │  Google Gemini     │
     │ (Prisma)  │          │  (fallback)        │
     └───────────┘          └────────────────────┘
           │
     ┌─────▼─────┐
     │  Redis 7  │
     │ Job Queue │
     │ + Cache   │
     └───────────┘
```

---

## Core Modules & Business Logic

### Reading Module — Instant Auto-Grading

The Reading module provides a complete IELTS Reading practice experience with **instant feedback**.

**Supported Question Types:**
- Multiple Choice (MCQ)
- Short Answer
- Matching Headings
- True / False / Not Given
- Diagram Labeling
- Sentence Completion

**Workflow:**

```
Learner selects passage (filtered by CEFR level: A2 → C1)
    ↓
Timer starts → Learner answers all questions
    ↓
POST /api/reading/passages/:id/submit
    ↓
Server compares answers to answer_key (JSONB)
  - MCQ: exact match
  - Short answer: keyword matching
    ↓
Instant response: {
  score_pct: 75%,
  correct_count: 9,
  total_questions: 12,
  per_question: [
    { question_id, is_correct, user_answer, correct_answer, explanation }
  ]
}
```

**Key Features:**
- CEFR level filtering (A2, B1, B2, C1)
- Timer with `timed_out` tracking
- Per-question explanations (authored by admin)
- Reading history with accuracy trends

---

### Writing Module — AI-Powered Scoring Pipeline

The most technically complex module. Uses an **async job queue** to offload AI scoring, keeping the API responsive.

**IELTS 4-Criteria Scoring:**

| Criterion | Code | Description |
|-----------|------|-------------|
| Task Response | TR | How well the essay addresses the prompt |
| Coherence & Cohesion | CC | Logical flow, paragraphing, linking |
| Lexical Resource | LR | Vocabulary range, accuracy, sophistication |
| Grammatical Range & Accuracy | GRA | Sentence structure variety, error frequency |

**Scoring Pipeline (Async):**

```
Client POST /api/writing/prompts/:id/submit
  ↓
WritingService.submitEssay()
  ├── Validate word count >= min_words
  ├── Check daily rate limit (10/day per user)
  └── Create submission (status: PENDING)
  ↓
ScoringProducerService.enqueue()
  ├── Add job to BullMQ 'writing-scoring' queue
  ├── Priority: 1 (premium GPT-4o) | 5 (cheap GPT-4o-mini)
  └── Return 202 { submission_id }
  ↓
Client polls GET /submissions/:id — OR — listens via SSE /events
  ↓
ScoringConsumer (BullMQ worker, concurrency: 3)
  ├── Fetch submission + prompt from DB
  ├── Build system prompt (IELTS rubric instructions)
  ├── Call LLM:
  │   ├── Primary: OpenAI (GPT-4o-mini or GPT-4o)
  │   └── Fallback: Google Gemini (if OpenAI fails)
  ├── Parse JSON response:
  │   {
  │     TR: 6.5, CC: 7.0, LR: 6.0, GRA: 6.5,
  │     overall: 6.5,
  │     summary: "...",
  │     strengths: ["...", "..."],
  │     improvements: ["...", "..."]
  │   }
  ├── Validate against strict JSON schema
  └── Update submission: scores, feedback, status → DONE
  ↓
Redis publishes event → SSE pushes to client
  ↓
Client displays detailed feedback panel
```

**Resilience Features:**
- 3 retry attempts with exponential backoff (5s base)
- Dual LLM provider (OpenAI primary, Gemini fallback)
- Schema validation ensures consistent scoring format
- Rate limiting prevents cost overruns (10 submissions/day)
- Priority queue: premium users get scored faster

---

### Classroom Module — Instructor-Led Learning

Instructors can create virtual classrooms and organize structured learning paths.

```
Classroom
  ├── Invite Code (8-char, shareable / QR code)
  ├── Members (teacher + students)
  ├── Topics (chapters/units)
  │   ├── Lesson 1 (text content)
  │   ├── Lesson 2 (linked Reading passage)
  │   ├── Lesson 3 (linked Writing prompt)
  │   └── Lesson 4 (video URL)
  └── Announcements (class-wide notifications)
```

**Key Features:**
- Join via invite code
- Hierarchical content: Topic > Lesson
- Lessons can link to existing Reading passages or Writing prompts
- `allow_submit` flag: enables learner submission per lesson
- Teacher can view all learner submissions and progress
- Class-wide announcements

---

### Admin Module — Content Management & Analytics

Full content lifecycle management with version tracking.

**Content CRUD:**
- Passages (title, body, CEFR level, topic tags, publish status)
- Questions (per passage — type, options, answer_key, explanation)
- Writing Prompts (task type, title, level, min_words)

**Content Versioning:**
- Every create/update/publish/unpublish/delete action is logged
- Tracks: editor ID, timestamp, diff (changes JSON)
- Full audit trail for compliance

**Admin Dashboard:**
- Total submissions (reading + writing)
- Popular passages/prompts (by submission count)
- Failure rates and error patterns
- User management (search, role changes)

---

## Authentication & Authorization

```
Registration → Email + Password (bcrypt hash)
    ↓
Login → POST /api/auth/login
    ↓
JWT Generation:
  - Access Token (15 min TTL): { sub, email, role }
  - Refresh Token (7 day TTL): httpOnly cookie
    ↓
Protected Routes → @UseGuards(JwtAuthGuard)
    ↓
RBAC → @Roles('admin') decorator checks JWT role
    ↓
Token Refresh → Axios interceptor catches 401 → silent refresh
```

**Role-Based Access Control (RBAC):**

| Feature | Learner | Instructor | Admin |
|---------|:-------:|:----------:|:-----:|
| Browse published passages/prompts | Y | Y | Y |
| Submit reading answers | Y | - | - |
| Submit writing essays | Y | - | - |
| View own submission history | Y | - | - |
| Create/manage classrooms | - | Y | Y |
| View learner submissions | - | Y | Y |
| CRUD content (passages, prompts) | - | - | Y |
| Publish/unpublish content | - | - | Y |
| Manage users & roles | - | - | Y |

---

## Database Schema

**14 tables** designed with Prisma 6 ORM on PostgreSQL 15:

```
Users & Auth
  └── users (email, password_hash, role, language, theme)
  └── content_versions (audit trail for all edits)

Content
  └── passages (title, body, CEFR level, status)
  └── questions (type, options, answer_key, explanation)
  └── prompts (task_type, title, level, min_words)
  └── topic_tags, collections

Submissions
  └── submissions_reading (answers JSONB, score_pct, duration_sec, timed_out)
  └── submissions_writing (content, scores JSONB, feedback JSONB, processing_status)

Classrooms
  └── classrooms (invite_code, owner_id, max_members)
  └── classroom_members (role: teacher/student)
  └── topics (hierarchical content structure)
  └── lessons (content_type, linked_entity_id, allow_submit)
  └── lesson_submissions, announcements

System
  └── notifications (type, title, message, link, is_read)
  └── source_documents, import_jobs (async content import)
```

---

## Technology Stack

### Frontend
- **Framework**: Next.js 16 (App Router, SSR/SSG, Turbopack)
- **Language**: TypeScript 5.x (strict mode)
- **Styling**: Tailwind CSS 4 with CSS variables (dark/light theme)
- **Server State**: TanStack React Query v5 (caching, auto-refetch)
- **HTTP**: Axios with JWT interceptor + silent token refresh
- **i18n**: Custom provider (Vietnamese/English, real-time switching)
- **Components**: Lucide React icons, react-hot-toast notifications

### Backend
- **Framework**: NestJS 10 (modular architecture, DI)
- **Language**: TypeScript 5.x (strict mode)
- **ORM**: Prisma 6 (type-safe queries, migrations, seed scripts)
- **Auth**: JWT (access 15min + refresh 7d) with RBAC guards
- **Queue**: BullMQ v5 + Redis 7 (async scoring jobs)
- **LLM**: OpenAI API (GPT-4o-mini / GPT-4o) + Google Gemini (fallback)
- **Validation**: class-validator + DTOs for all inputs
- **File Upload**: Multer with custom upload service

### Infrastructure
- **Database**: PostgreSQL 15 (Docker)
- **Cache/Queue**: Redis 7 Alpine (Docker)
- **Containerization**: Docker Compose
- **Testing**: Jest + Supertest

---

## API Design

**15+ REST endpoints** organized by domain module:

| Module | Endpoints | Key Operations |
|--------|-----------|----------------|
| Auth | `/api/auth/*` | Register, Login, Refresh, Profile |
| Reading | `/api/reading/*` | List passages, Get passage + questions, Submit answers, History |
| Writing | `/api/writing/*` | List prompts, Submit essay (async), Get submission + feedback, SSE events |
| Dashboard | `/api/dashboard/*` | Role-specific stats and progress |
| Admin | `/api/admin/*` | CRUD passages/questions/prompts, Publish/unpublish, User management, Analytics |
| Instructor | `/api/instructor/*` | Learner list, View all submissions |
| Classroom | `/api/classrooms/*` | Create, Join (invite code), Topics/Lessons CRUD, Announcements |
| Notifications | `/api/notifications/*` | List, Mark as read |

---

## Key Technical Decisions

| Decision | Why |
|----------|-----|
| **NestJS** over Express | Modular architecture, built-in DI, guards, decorators — scales better for multi-module apps |
| **Prisma** over TypeORM | Type-safe queries, excellent migration system, better DX with auto-generated types |
| **BullMQ** for scoring queue | Reliable job queue with priority, retry, and backoff — critical for async LLM calls |
| **Dual LLM providers** | OpenAI primary + Gemini fallback ensures scoring availability even during outages |
| **SSE** over WebSocket for scoring | Simpler protocol for unidirectional server-to-client updates (score status) |
| **JSONB** for scores/feedback | Flexible schema for structured AI responses without extra tables |
| **Next.js App Router** | Server-side rendering for SEO, file-based routing, React Server Components |
| **TanStack Query** over Redux | Server state management with built-in caching — no boilerplate reducers needed |

---

## Development Workflow

The project follows a **PRD-first development process**:

```
1. Business Idea & Requirements Analysis
   ↓
2. Wireframes & Data Flow Design
   ↓
3. Product Requirements Document (PRD) — 18 detailed documents
   covering executive summary, personas, functional requirements,
   data models, API specs, and more
   ↓
4. Implementation Plan (Sprint-based)
   ↓
5. Code Implementation with AI-assisted development
   ↓
6. Review & Iteration
```

**Comprehensive Documentation:**
- 18-document PRD suite covering every aspect of the system
- Design specs for complex features (e.g., Writing Scoring Pipeline)
- Implementation plans per sprint
- Getting Started guide for developer onboarding

---

## Challenges & Solutions

| Challenge | Solution |
|-----------|----------|
| AI scoring consistency | Rubric-based system prompts + strict JSON schema validation + response parsing |
| LLM response latency (10–30s) | Async BullMQ queue + SSE real-time updates — API returns 202 immediately |
| Cost control for AI calls | Rate limiting (10/day), cheap model as default (GPT-4o-mini), premium as opt-in |
| Multi-question type grading | JSONB answer storage + per-type comparison logic (exact match, keyword match) |
| Content quality assurance | Draft/Published workflow + version tracking + admin-only publish rights |
| Dual language support | Custom i18n provider with JSON translation files, real-time language switching |

---

## Roadmap

**Completed:**
- Project scaffolding (monorepo, Docker, Prisma)
- Authentication (JWT, RBAC, token refresh)
- Reading module (passages, questions, auto-grading, history)
- Writing module (prompts, submission, async scoring pipeline)
- Admin CRUD (passages, questions, prompts, users)
- Classroom management (create, topics, lessons, members)
- Instructor dashboard

**In Progress:**
- AI Writing Scoring Pipeline optimization (priority queue, SSE)
- Dashboard analytics (trends, cohort stats)

**Planned:**
- Listening & Speaking modules
- OAuth2 (Google login)
- Content import via document upload (AI-powered extraction)
- Mobile-responsive optimization
- Instructor score override & comments
- Advanced analytics (cohort comparison, ML recommendations)
