# Synoptiq Documentation

Public technical documentation for **[Synoptiq](https://usesynoptiq.com)** — a workflow automation platform that connects Gmail, Google Calendar, GitHub, and workspace search into one intelligent dashboard.

> **Note:** Application source code (backend + frontend) is maintained in **private repositories**.  
> This public repo contains architecture and API documentation only — no secrets, no source code.

| | |
|---|---|
| **Live application** | [usesynoptiq.com](https://usesynoptiq.com) |
| **API** | [api.abhiram.tech](https://api.abhiram.tech) |
| **Swagger UI** | [api.abhiram.tech/swagger-ui](https://api.abhiram.tech/swagger-ui/index.html) |
| **Docs repo** | [github.com/abhiramaab/synoptiq-docs](https://github.com/abhiramaab/synoptiq-docs) |
| **Author** | [Abhirama B](https://github.com/abhiramaab) |
| **Contact** | abhiram.b@icloud.com |

---

## What is Synoptiq?

Synoptiq helps professionals automate inbox and developer workflows:

- **Google OAuth2** sign-in with JWT sessions
- **Gmail** — sync, search, summarization, watchlists, notifications, attachment downloads
- **Google Calendar** — today's events, upcoming schedule, search, create events from chat, daily summaries
- **GitHub** — OAuth linking, repo tracking, commits, PRs, natural-language counts and date-range queries
- **Crawls** — unified natural-language search across email, attachments, GitHub, and calendar
- **Daily Brief** — morning overview of inbox, meetings, and GitHub activity
- **AI Workspace** — conversational assistant with intent routing, agent tools, and fast structured replies
- **Reply & compose drafts** — generated emails with approval workflow
- **Subscriptions** — Edge & Pinnacle plans via Razorpay

---

## Recent Updates (July 2026)

| Area | What shipped |
|------|----------------|
| **Domain** | Production frontend moved to `usesynoptiq.com` (Vercel); API remains `api.abhiram.tech` (EC2) |
| **Agent platform** | `chat/agent/*` — router, planner, parallel tool executor, memory, response cache |
| **Fast chat** | Structured intents skip LLM; greetings instant; tool results returned directly when possible |
| **Calendar** | `GET /calendar/events/upcoming`, `POST /calendar/events/create`, extended `/calendar/status` |
| **GitHub chat** | `GithubQueryParser` for counts and date ranges; parallel repo fetch with 2-min cache |
| **Automations UI** | Live connect-state toolbars; real calendar upcoming events (no static placeholders) |
| **Legal pages** | Updated Privacy Policy and Terms of Service (July 31, 2026) |
| **AI models** | OpenAI enum → API slug mapping (`GPT_4_1_MINI` → `gpt-4.1-mini`) |
| **Database** | `agent_session_states`, `semantic_memories` tables for agent platform |

---

## Tech Stack

| Layer | Technologies |
|-------|-------------|
| **Backend** | Java 21, Spring Boot 3.5, Spring Security, JPA/Hibernate, PostgreSQL |
| **Frontend** | React 18, React Router 7, TanStack Query, Tailwind CSS, shadcn/ui |
| **Auth** | JWT, Google OAuth2, GitHub OAuth |
| **Integrations** | Gmail API, Google Calendar API, GitHub REST API |
| **Deploy** | Docker (EC2 API), Vercel (frontend), Neon PostgreSQL |

---

## Documentation Index

| Document | Description |
|----------|-------------|
| [Overview](./OVERVIEW.md) | One-page summary for recruiters and quick onboarding |
| [Architecture](./ARCHITECTURE.md) | System design, agent platform, chat routing, integrations |
| [API Reference](./API.md) | REST endpoints grouped by domain |
| [Authentication](./AUTHENTICATION.md) | JWT, Google OAuth, GitHub OAuth flows |
| [Database](./DATABASE.md) | Entity relationships and schema overview |
| [Frontend](./FRONTEND.md) | React app structure, automations UI, workspace chat |
| [Deployment](./DEPLOYMENT.md) | Production architecture and domain setup (no secrets) |
| [Sync guide](./SYNC.md) | How to keep this repo aligned with private backend docs |

---

## Architecture at a Glance

```
┌─────────────────────┐         HTTPS / REST          ┌──────────────────────────┐
│  React Frontend     │  ◄──────────────────────────► │  Spring Boot API         │
│  usesynoptiq.com    │      JWT Bearer Auth          │  api.abhiram.tech        │
│  (Vercel)           │                               │  (EC2 + Docker)          │
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
