# Synoptiq Documentation

Public technical documentation for **[Synoptiq](https://synoptiq.abhiram.tech)** — a workflow automation platform that connects Gmail, Google Calendar, GitHub, and workspace search into one intelligent dashboard.

> **Note:** Application source code (backend + frontend) is maintained in **private repositories**.  
> This public repo contains architecture and API documentation only — no secrets, no source code.

| | |
|---|---|
| **Live application** | [synoptiq.abhiram.tech](https://synoptiq.abhiram.tech) |
| **API** | [api.abhiram.tech](https://api.abhiram.tech) |
| **Swagger UI** | [api.abhiram.tech/swagger-ui](https://api.abhiram.tech/swagger-ui/index.html) |
| **Author** | [Abhirama B](https://github.com/abhiramaab) |
| **Contact** | abhiram.b@icloud.com |

---

## What is Synoptiq?

Synoptiq helps professionals automate inbox and developer workflows:

- **Google OAuth2** sign-in with JWT sessions
- **Gmail** — sync, search, summarization, watchlists, notifications
- **Google Calendar** — today's events, search, daily summaries
- **GitHub** — OAuth linking, repo tracking, commits, PRs, dev summaries
- **Crawls** — unified natural-language search across email, attachments, GitHub, and calendar
- **Daily Brief** — morning overview of inbox, meetings, and GitHub activity
- **AI Workspace** — conversational assistant with intent routing
- **Reply & compose drafts** — generated emails with approval workflow
- **Subscriptions** — Edge & Pinnacle plans via Razorpay

---

## Tech Stack

| Layer | Technologies |
|-------|-------------|
| **Backend** | Java 21, Spring Boot 3.5, Spring Security, JPA/Hibernate, PostgreSQL |
| **Frontend** | React 18, React Router 7, TanStack Query, Tailwind CSS, shadcn/ui |
| **Auth** | JWT, Google OAuth2, GitHub OAuth |
| **Integrations** | Gmail API, Google Calendar API, GitHub REST API |
| **Deploy** | Docker, AWS EC2, Neon PostgreSQL |

---

## Documentation Index

| Document | Description |
|----------|-------------|
| [Overview](./OVERVIEW.md) | One-page summary for recruiters and quick onboarding |
| [Architecture](./ARCHITECTURE.md) | System design, modules, schedulers, integrations |
| [API Reference](./API.md) | REST endpoints grouped by domain |
| [Authentication](./AUTHENTICATION.md) | JWT, Google OAuth, GitHub OAuth flows |
| [Database](./DATABASE.md) | Entity relationships and schema overview |
| [Frontend](./FRONTEND.md) | React app structure, routing, services |
| [Deployment](./DEPLOYMENT.md) | High-level production architecture (no secrets) |

---

## Architecture at a Glance

```
┌─────────────────────┐         HTTPS / REST          ┌──────────────────────────┐
│  React Frontend     │  ◄──────────────────────────► │  Spring Boot API         │
│  (CRA + Tailwind)   │      JWT Bearer Auth          │  Java 21                 │
└─────────────────────┘                               └────────────┬─────────────┘
                                                                   │
                    ┌──────────────────────────────────────────────┼──────────────┐
                    ▼                                              ▼              ▼
            ┌───────────────┐                              ┌───────────────┐  ┌──────────┐
            │  PostgreSQL   │                              │  Google APIs  │  │  GitHub  │
            │  (Neon)       │                              │  Gmail·Cal    │  │  API     │
            └───────────────┘                              └───────────────┘  └──────────┘
```

---

## Source Code Access

Backend and frontend repositories are **private**. For technical interviews, architecture walkthroughs, or collaborator access, contact:

**abhiram.b@icloud.com**

---

## License

Documentation © 2026 Abhirama B. All rights reserved.  
Synoptiq application source code is proprietary and not included in this repository.
