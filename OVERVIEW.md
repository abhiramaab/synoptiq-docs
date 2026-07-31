# Synoptiq — Project Overview

A one-page summary of Synoptiq for recruiters, hiring managers, and technical reviewers.

---

## Elevator Pitch

**Synoptiq** is a production-deployed workflow automation platform built with **Java Spring Boot** and **React**. It connects a user's Gmail, Google Calendar, and GitHub into a single workspace where they can search, summarize, draft replies, schedule meetings from chat, and get a daily brief — all secured with OAuth2 and JWT.

**Live demo:** [usesynoptiq.com](https://usesynoptiq.com)

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
| **Gmail integration** | OAuth token management, scheduled inbox sync (every 5 min), natural-language search, summarization, attachment downloads |
| **Calendar integration** | Shared Google token, event listing, NL date parsing, create events from chat, upcoming schedule API |
| **GitHub integration** | Separate OAuth flow, encrypted token storage, repo dashboard, NL count/date queries, parallel activity fetch |
| **AI Workspace** | Intent-based chat routing, agent tool platform, fast paths for greetings and structured data |
| **REST API design** | DTO layer, validation, pagination, modular controllers per domain |
| **Database** | PostgreSQL with 26+ JPA entities, user-scoped data, encrypted sensitive fields |
| **Background jobs** | Spring schedulers for Gmail sync and daily summaries |
| **Frontend** | React SPA on Vercel, protected routes, TanStack Query, live automations dashboards |
| **Deployment** | Docker on AWS EC2 (API), Vercel (frontend), Neon PostgreSQL, custom domain |
| **Billing** | Razorpay subscription integration (Edge / Pinnacle plans) |

---

## Architecture Style

- **Modular monolith** backend (not microservices) — single deployable JAR with domain-separated packages
- **Separate frontend** SPA on Vercel communicating via REST + JWT
- **Layered design:** Controller → Service → Repository → Entity
- **Integration isolation:** Gmail, Calendar, GitHub each in dedicated packages
- **Agent layer:** `chat/agent/*` for tool orchestration on top of intent routing

---

## Core User Flows

### 1. Sign in
User clicks "Sign in with Google" on `usesynoptiq.com` → redirect to `api.abhiram.tech/oauth2/authorization/google` → OAuth consent → JWT issued → redirected to `/oauth-success?token=...` on frontend.

### 2. Daily Brief
User opens `/brief` → backend aggregates today's emails, calendar events, and GitHub activity → AI-generated summary displayed.

### 3. Crawls (Search)
User types natural language query on dashboard → backend searches emails, attachments, GitHub, and calendar → results with preview panel.

### 4. Workspace chat
User asks in natural language → intent router picks Gmail, GitHub, Calendar, or agent path → structured data returned directly when possible → LLM used only when a narrative is needed.

### 5. Calendar create from chat
User says "schedule team sync tomorrow at 3pm" → `CALENDAR_CREATE` intent → AI extracts event details → event created in Google Calendar.

### 6. Reply draft approval
Background job or user trigger generates reply draft → user reviews in workspace → approve/reject → sent via Gmail API.

---

## Scale & Scope

| Metric | Approximate |
|--------|-------------|
| Backend Java classes | 320+ |
| REST controllers | 21+ |
| JPA entities | 26+ |
| Agent tools | 10 (Gmail, GitHub, Calendar, Workspace, etc.) |
| API endpoint groups | 15+ domains |
| Frontend pages | 15+ routes |
| External integrations | Gmail, Calendar, GitHub, Razorpay, OpenAI |

---

## Production Domains

| Service | URL | Hosting |
|---------|-----|---------|
| Frontend | `https://usesynoptiq.com` | Vercel |
| API | `https://api.abhiram.tech` | AWS EC2 + Docker |
| Database | Neon PostgreSQL | Serverless |

> **Important:** Frontend and API are on **separate domains**. OAuth login always starts on `api.abhiram.tech`; the backend redirects back to `usesynoptiq.com/oauth-success` after success.

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
3. Agent platform and intent routing
4. Gmail sync scheduler and data model
5. One API endpoint traced end-to-end (controller → service → repository)
6. Deployment setup (Vercel + EC2 + Neon)

Contact: **abhiram.b@icloud.com**

---

## Related Docs

- [Architecture](./ARCHITECTURE.md) — full technical design
- [API Reference](./API.md) — all endpoints
- [Authentication](./AUTHENTICATION.md) — auth flows with diagrams
