<p align="center">
  <a href="https://usesynoptiq.com">
    <img src="https://raw.githubusercontent.com/abhiramaab/synoptiq-docs/main/assets/synoptiq-logo.svg" alt="Synoptiq" width="100" />
  </a>
</p>

<h1 align="center">Synoptiq Documentation</h1>

<p align="center">
  Intelligent Workspace Automation Platform &amp; Autonomous Agent Engine<br/>
  Orchestrates Gmail, Google Calendar, GitHub, and unified semantic search into an automated developer and productivity console.
</p>

<p align="center">
  <a href="https://usesynoptiq.com">
    <img src="https://img.shields.io/badge/Live_Application-usesynoptiq.com-7C3AED?style=for-the-badge&logo=vercel&logoColor=white" alt="Live Application" />
  </a>
  <a href="https://api.abhiram.tech/swagger-ui/index.html">
    <img src="https://img.shields.io/badge/OpenAPI_Spec-api.abhiram.tech-10B981?style=for-the-badge&logo=swagger&logoColor=white" alt="Swagger API" />
  </a>
  <a href="https://github.com/abhiramaab/synoptiq-docs">
    <img src="https://img.shields.io/badge/GitHub-Docs_Repository-181717?style=for-the-badge&logo=github&logoColor=white" alt="GitHub Docs" />
  </a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Java-21-ED8B00?style=flat-square&logo=openjdk&logoColor=white" alt="Java 21" />
  <img src="https://img.shields.io/badge/Spring_Boot-3.5-6DB33F?style=flat-square&logo=springboot&logoColor=white" alt="Spring Boot 3.5" />
  <img src="https://img.shields.io/badge/Spring_Security-OAuth2_+_JWT-6DB33F?style=flat-square&logo=springsecurity&logoColor=white" alt="Spring Security" />
  <img src="https://img.shields.io/badge/PostgreSQL-Neon_Cloud-4169E1?style=flat-square&logo=postgresql&logoColor=white" alt="PostgreSQL" />
  <img src="https://img.shields.io/badge/OpenAI-GPT--4.1--mini-412991?style=flat-square&logo=openai&logoColor=white" alt="OpenAI" />
  <img src="https://img.shields.io/badge/React-18-61DAFB?style=flat-square&logo=react&logoColor=black" alt="React 18" />
  <img src="https://img.shields.io/badge/TypeScript-5.x-3178C6?style=flat-square&logo=typescript&logoColor=white" alt="TypeScript" />
  <img src="https://img.shields.io/badge/TailwindCSS-3.x-06B6D4?style=flat-square&logo=tailwindcss&logoColor=white" alt="Tailwind CSS" />
  <img src="https://img.shields.io/badge/Razorpay-Billing-0C2340?style=flat-square&logo=razorpay&logoColor=white" alt="Razorpay" />
</p>

---

<details>
<summary><strong>Table of Contents</strong></summary>

- [Overview](#overview)
- [System Architecture](#system-architecture)
- [System Design & Core Modules](#system-design--core-modules)
  - [1. Autonomous Agent Engine: Router, Planner & Tool Executor](#1-autonomous-agent-engine-router-planner--tool-executor)
  - [2. Multi-Provider OAuth2 Security & Token Lifecycle](#2-multi-provider-oauth2-security--token-lifecycle)
  - [3. High-Throughput Email Ingestion & Incremental Sync](#3-high-throughput-email-ingestion--incremental-sync)
  - [4. Unified Cross-Platform Search (Crawls)](#4-unified-cross-platform-search-crawls)
  - [5. Automated Style Profiling & Draft Generation](#5-automated-style-profiling--draft-generation)
- [Documentation Index](#documentation-index)
- [Technology Stack](#technology-stack)
- [API Reference Summary](#api-reference-summary)
- [Production Deployment Topology](#production-deployment-topology)
- [Source Code & Access](#source-code--access)

</details>

---

## Overview

Synoptiq is an intelligent workspace coordination platform that connects inbox, schedule, and code management systems into a unified automation surface. Instead of manually switching across fragmented SaaS dashboards, users query, organize, summarize, and execute actions across Gmail, Google Calendar, and GitHub through an autonomous agent engine.

Key architectural highlights:

* **Modular Monolith Backend**: Java 21 and Spring Boot 3.5 architecture decoupling business domains while maintaining transactional consistency.
* **Deterministic Agent Routing**: Fast-path intent classifiers skip LLM calls for structured queries (greetings, counts, date-filtered lookups), cutting latency to sub-100ms.
* **Autonomous Parallel Tool Calling**: Complex multi-step instructions are decomposed into dependency DAGs and executed across integrated APIs concurrently.
* **Unified Semantic Crawls**: Single-query search indexing messages, thread attachments, upcoming meetings, pull requests, and commit logs.
* **Enterprise Security Invariants**: AES-256 encrypted refresh token storage, stateless HMAC JWT API verification, and strict user-scoped row tenancy.

---

## System Architecture

```text
 Client Web Application (React 18 + Vite / Vercel Edge)
                          │
                          ▼
            ┌───────────────────────────┐
            │   JwtAuthenticationFilter │ ──(Invalid Token)──► HTTP 401 Unauthorized
            └───────────────────────────┘
                          │ (Authenticated UserPrincipal)
                          ▼
 ┌────────────────────────────────────────────────────────┐
 │           SYNOPTIQ BACKEND ORCHESTRATOR                │
 │                                                        │
 │  ┌──────────────────────────────────────────────────┐  │
 │  │      Chat & Agent Platform (chat.agent.*)        │  │
 │  │  ├── Intent Classifier (Fast path vs LLM path)   │  │
 │  │  ├── Execution Planner (Dependency DAG)         │  │
 │  │  ├── Parallel Tool Executor (CompletableFuture)  │  │
 │  │  └── Semantic Memory & Short-Term Context Cache  │  │
 │  └──────────────────────────────────────────────────┘  │
 │                            │                           │
 │     ┌──────────────────────┼──────────────────────┐    │
 │     ▼                      ▼                      ▼    │
 │  Email Sync          Calendar Sync          GitHub Client│
 │  & Summaries         & Event Planner        & PR Parser │
 └────────────────────────────────────────────────────────┘
       │                      │                      │
       ▼                      ▼                      ▼
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│  Gmail API   │       │ Google Cal   │       │ GitHub REST  │
│  (OAuth2)    │       │ API (OAuth2) │       │ API (OAuth2) │
└──────────────┘       └──────────────┘       └──────────────┘
       │
       ▼
┌────────────────────────────────────────────────────────┐
│             NEON POSTGRESQL PERSISTENCE                │
│  - Encrypted OAuth Tokens (AES-256)                    │
│  - Email Threads, Attachments & Watchlists             │
│  - Agent Session States & Semantic Memory              │
│  - Razorpay Subscriptions (Edge & Pinnacle Tiers)      │
└────────────────────────────────────────────────────────┘
```

---

## System Design & Core Modules

### 1. Autonomous Agent Engine: Router, Planner & Tool Executor
* **Intent Classification**: Evaluates incoming queries to determine whether an instruction can be fulfilled deterministically (e.g., upcoming meetings, unread email counts, GitHub repository lists) or requires LLM synthesis.
* **Fast-Chat Bypass**: Structured commands bypass LLM completion overhead entirely, achieving sub-100ms response times.
* **Parallel Tool Executor**: Utilizes Java 21 virtual threads and `CompletableFuture` to fetch data concurrently across disparate external providers (e.g., retrieving calendar conflicts and GitHub PR reviews simultaneously).
* **Stateful Context Cache**: Tracks session context and semantic memories (`agent_session_states`, `semantic_memories`) to preserve conversational continuity.

### 2. Multi-Provider OAuth2 Security & Token Lifecycle
* **AES-256 Symmetric Encryption**: Access and refresh tokens for Google and GitHub are encrypted before persisting to PostgreSQL.
* **Automatic Token Refresh**: Transparently exchanges expired OAuth access tokens with upstream providers without user intervention or session drops.
* **Dual Auth Boundary**: Stateless HMAC-SHA256 JWT tokens authenticate user requests to the backend REST API; OAuth sessions are isolated exclusively to initial provider handshake flows.

### 3. High-Throughput Email Ingestion & Incremental Sync
* **Change-Vector Sync**: Synchronizes mailbox states via history tokens and incremental deltas rather than repetitive bulk downloads.
* **Background Worker Processing**: Dedicated workers process attachment downloads, generate structured thread summaries, and maintain user watchlist alerts.

### 4. Unified Cross-Platform Search (Crawls)
* **Consolidated Search Parser**: Resolves complex natural language queries across email messages, thread attachments, GitHub commits, pull requests, and calendar invites.
* **Multi-Domain Relevance**: Normalizes query scores across disparate data structures to present a chronological, prioritized feed.

### 5. Automated Style Profiling & Draft Generation
* **Writing Style Extraction**: Analyzes historical outbound emails to construct persona heuristics (tone, sign-offs, typical length, greeting formalities).
* **Supervised Draft Composition**: Generates contextual draft replies that preserve user voice while enforcing human-in-the-loop review before sending.

---

## Documentation Index

Detailed engineering documentation is organized into domain-specific guides:

| Document | Focus & Coverage |
| :--- | :--- |
| [Overview](./OVERVIEW.md) | High-level capabilities, value proposition, and user experience summary |
| [Architecture](./ARCHITECTURE.md) | Monolith domain boundaries, request lifecycle, agent engine DAG, and design patterns |
| [API Reference](./API.md) | Exhaustive REST endpoint contracts across Auth, Chat, Email, Calendar, GitHub, and Billing |
| [Authentication](./AUTHENTICATION.md) | In-depth OAuth2 handshake workflows, JWT issuance, and AES token encryption |
| [Database](./DATABASE.md) | Relational schema diagrams, table constraints, entity mappings, and indices |
| [Frontend](./FRONTEND.md) | React 18 component hierarchies, TanStack Query cache invalidation, and UI architecture |
| [Deployment](./DEPLOYMENT.md) | Production topology, Docker packaging on AWS EC2, and Vercel edge deployment |
| [Sync Guide](./SYNC.md) | Protocols for syncing public documentation with private upstream development repositories |

---

## Technology Stack

| Layer | Technologies |
| :--- | :--- |
| Backend Core | Java 21, Spring Boot 3.5, Spring MVC, Maven |
| Security & Auth | Spring Security 6, Stateless JWT, Google OAuth2, GitHub OAuth |
| Relational Storage | PostgreSQL (Neon Serverless), Hibernate, Spring Data JPA |
| AI Orchestration | OpenAI GPT-4.1-mini, Custom Intent Router, Parallel Tool Executor |
| External Integrations | Google Workspace (Gmail & Calendar APIs), GitHub REST API |
| Billing & Payments | Razorpay Subscription Engine (Webhook verification) |
| Frontend Console | React 18, React Router 7, TanStack Query, Tailwind CSS, shadcn/ui |
| Hosting & Cloud | AWS EC2 (Dockerized Backend), Vercel (Frontend Edge) |

---

## API Reference Summary

### Authentication & Users
* `POST /api/auth/register` - Create account with email & password
* `POST /api/auth/login` - Authenticate credentials and receive JWT
* `GET /api/users/me` - Fetch profile, storage quotas, and connected services

### Intelligent Workspace Chat & Agent
* `POST /api/chat/messages` - Dispatch prompt to agent router and execution planner
* `GET /api/chat/conversations` - Retrieve user conversation history
* `DELETE /api/chat/conversations/{id}` - Clear conversation thread

### Unified Email & Gmail
* `GET /api/email/threads` - List indexed threads with unread and category filters
* `POST /api/email/sync` - Trigger incremental delta synchronization
* `POST /api/compose/draft` - Generate AI reply draft with style profile matching

### Calendar & GitHub Integrations
* `GET /api/calendar/events/upcoming` - Retrieve upcoming meetings and agenda
* `POST /api/calendar/events/create` - Create event directly from conversational intent
* `GET /api/github/repos` - List linked repositories with commit metrics and open PRs

---

## Production Deployment Topology

```text
       DNS: usesynoptiq.com                      DNS: api.abhiram.tech
               │                                           │
               ▼                                           ▼
    ┌──────────────────────┐                    ┌──────────────────────┐
    │     Vercel Edge      │                    │     AWS EC2 Host     │
    │  React 18 SPA Build  │                    │ Dockerized Spring 3.5│
    └──────────────────────┘                    └──────────┬───────────┘
               │                                           │
               └──────────── HTTPS / REST + JWT ───────────┘
                                                           │
                                                           ▼
                                                ┌──────────────────────┐
                                                │   Neon PostgreSQL    │
                                                │ (Serverless Cluster) │
                                                └──────────────────────┘
```

---

## Source Code & Access

Application source repositories for both backend and frontend are maintained as private codebases. For architecture walkthroughs, technical evaluations, or engineering inquiries, contact:

* **Author**: [Abhirama B](https://github.com/abhiramaab)
* **Portfolio**: [portfolio.abhiram.tech](https://portfolio.abhiram.tech)
* **Email**: abhiram.b@icloud.com

---

<p align="center">
  Built by <a href="https://github.com/abhiramaab">Abhirama</a> · Live at <a href="https://portfolio.abhiram.tech">portfolio.abhiram.tech</a>
</p>
