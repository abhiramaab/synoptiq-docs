# Synoptiq — Project Overview

A one-page summary of Synoptiq for recruiters, hiring managers, and technical reviewers.

---

## Elevator Pitch

**Synoptiq** is a production-deployed workflow automation platform built with **Java Spring Boot** and **React**. It connects a user's Gmail, Google Calendar, and GitHub into a single workspace where they can search, summarize, draft replies, and get a daily brief — all secured with OAuth2 and JWT.

**Live demo:** [synoptiq.abhiram.tech](https://synoptiq.abhiram.tech)

---

## Built By

**Abhirama B** — Java Backend Developer  
GitHub: [github.com/abhiramaab](https://github.com/abhiramaab)  
Email: abhiram.b@icloud.com

---

## Key Technical Achievements

| Area | What was built |
|------|----------------|
| **Authentication** | Google OAuth2 login, JWT API auth, GitHub account linking, BCrypt for local users |
| **Gmail integration** | OAuth token management, scheduled inbox sync (every 5 min), natural-language search, summarization |
| **Calendar integration** | Shared Google token, event listing, NL date parsing, daily meeting summaries |
| **GitHub integration** | Separate OAuth flow, encrypted token storage, repo dashboard, daily dev summaries |
| **REST API design** | DTO layer, validation, pagination, modular controllers per domain |
| **Database** | PostgreSQL with 20+ JPA entities, user-scoped data, encrypted sensitive fields |
| **Background jobs** | Spring schedulers for Gmail sync and daily summaries |
| **Frontend** | React SPA with protected routes, TanStack Query, service-layer API clients |
| **Deployment** | Docker on AWS EC2, Neon PostgreSQL, HTTPS API + static frontend |
| **Billing** | Razorpay subscription integration (Edge / Pinnacle plans) |

---

## Architecture Style

- **Modular monolith** backend (not microservices) — single deployable JAR with domain-separated packages
- **Separate frontend** SPA communicating via REST + JWT
- **Layered design:** Controller → Service → Repository → Entity
- **Integration isolation:** Gmail, Calendar, GitHub each in dedicated packages

---

## Core User Flows

### 1. Sign in
User clicks "Sign in with Google" → OAuth consent (Gmail + Calendar scopes) → JWT issued → redirected to dashboard.

### 2. Daily Brief
User opens `/brief` → backend aggregates today's emails, calendar events, and GitHub activity → AI-generated summary displayed.

### 3. Crawls (Search)
User types natural language query on dashboard → backend searches emails, attachments, GitHub, and calendar → results with preview panel.

### 4. Reply draft approval
Background job or user trigger generates reply draft → user reviews in workspace → approve/reject → sent via Gmail API.

---

## Scale & Scope

| Metric | Approximate |
|--------|-------------|
| Backend Java classes | 250+ |
| REST controllers | 21 |
| JPA entities | 24 |
| API endpoint groups | 15+ domains |
| Frontend pages | 15+ routes |
| External integrations | Gmail, Calendar, GitHub, Razorpay, OpenAI |

---

## What's Public vs Private

| Public | Private |
|--------|---------|
| This documentation repo | Backend source code |
| Live application (demo) | Frontend source code |
| Swagger API docs (when deployed) | Environment variables & secrets |
| Architecture descriptions | Database credentials |

---

## Interview Walkthrough Offer

Happy to provide a **15–20 minute live walkthrough** covering:

1. System architecture diagram
2. OAuth2 + JWT authentication flow
3. Gmail sync scheduler and data model
4. One API endpoint traced end-to-end (controller → service → repository)
5. Deployment setup (Docker + EC2 + Neon)

Contact: **abhiram.b@icloud.com**

---

## Related Docs

- [Architecture](./ARCHITECTURE.md) — full technical design
- [API Reference](./API.md) — all endpoints
- [Authentication](./AUTHENTICATION.md) — auth flows with diagrams
