# Architecture

Synoptiq is a **modular monolith**: a single Spring Boot application with clearly separated packages per domain. The React frontend is a separate deployable that communicates exclusively via REST + JWT.

---

## Design Principles

1. **Layered architecture** — Controller → Service → Repository → Entity
2. **DTO boundary** — API responses never expose entities directly
3. **Integration isolation** — Gmail, Calendar, and GitHub live under `integration/`
4. **Stateless API auth** — JWT on every request after login; OAuth sessions only during Google redirect
5. **User-scoped data** — every query filters by authenticated `User`

---

## High-Level Component Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           SYNOPTIQ BACKEND (Monolith)                        │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │   auth/     │  │   chat/     │  │   email/    │  │   behavior/         │ │
│  │   user/     │  │   compose/  │  │   search/   │  │   payment/          │ │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘ │
│         │                │                │                     │            │
│  ┌──────┴────────────────┴────────────────┴─────────────────────┴──────────┐ │
│  │                        Service Layer                                     │ │
│  └──────┬───────────────────────────────────────────────────────────────────┘ │
│         │                                                                      │
│  ┌──────┴───────────────────────────────────────────────────────────────────┐ │
│  │  integration/gmail · integration/calendar · integration/github · ai/     │ │
│  └──────┬───────────────────────────────────────────────────────────────────┘ │
│         │                                                                      │
│  ┌──────┴──────┐                    ┌──────────────┐                          │
│  │  JPA Repos  │◄──────────────────►│  PostgreSQL  │                          │
│  └─────────────┘                    └──────────────┘                          │
└─────────────────────────────────────────────────────────────────────────────┘
         ▲                              ▲                    ▲
         │ REST + JWT                   │                    │
┌────────┴────────┐            ┌────────┴────────┐  ┌───────┴────────┐
│ React Frontend  │            │  Google APIs    │  │  GitHub API    │
└─────────────────┘            └─────────────────┘  └────────────────┘
```

---

## Package Overview

| Package | Responsibility |
|---------|----------------|
| `auth` | Email/password register and login; issues JWT |
| `security` | `JwtService`, `JwtAuthenticationFilter` |
| `oauth` | Google OIDC login, token persistence to `gmail_tokens` |
| `user` | Profile (`GET/PATCH /api/users/me`) |
| `email` | Email entity, sync, search, summaries, attachments, notifications |
| `integration.gmail` | Low-level Gmail API client and dashboard |
| `integration.calendar` | Calendar events, search, daily summary |
| `integration.github` | GitHub OAuth, repo sync, dashboard, summaries |
| `chat` | Conversations, messages, intent routing to tools |
| `compose` | AI-assisted outbound email drafts and attachments |
| `behavior` | User writing-style profile, reply drafts, quotas |
| `search` | Unified search (Crawls), saved searches, history |
| `payment` | Razorpay subscription create/verify/cancel |
| `ai` | Model selection, OpenAI completion client |
| `config` | Security, CORS, OpenAPI, OAuth resolver |
| `common` | Exception handling, encryption converter |

---

## Request Lifecycle

Typical authenticated API request:

```
Client
  │  Authorization: Bearer <jwt>
  ▼
JwtAuthenticationFilter
  │  validate token → load User by email
  ▼
SecurityContext (authenticated)
  ▼
@RestController
  ▼
@Service (business logic, external API calls)
  ▼
@Repository (JPA)
  ▼
PostgreSQL
```

Validation uses Jakarta Bean Validation on request DTOs. Errors are handled by a global `@ControllerAdvice` in `common`.

---

## Authentication Architecture

Three paths into the system:

| Method | Entry | Result |
|--------|-------|--------|
| **Register / Login** | `POST /api/auth/register` or `/login` | JWT in JSON response |
| **Google OAuth** | `/oauth2/authorization/google` | Redirect to frontend with `?token=<jwt>` |
| **GitHub link** | `GET /api/github/oauth/authorize-url` (requires existing JWT) | Stores GitHub token; does not replace login |

See [AUTHENTICATION.md](./AUTHENTICATION.md) for sequence diagrams.

---

## Integration Layer

### Gmail

- OAuth tokens stored in `gmail_tokens` (one row per user)
- `GmailService` wraps Google Gmail API
- `GmailSyncScheduler` runs every **5 minutes** to pull new messages
- Emails persisted in `emails` with attachments in `email_attachments`

### Google Calendar

- Uses the **same Google OAuth token** as Gmail
- Scopes: `calendar.readonly`
- `CalendarService` lists events, supports natural-language date parsing
- `CalendarSummaryService` generates daily meeting summaries

### GitHub

- **Separate OAuth flow** from Google login
- Access token encrypted at rest (`APP_ENCRYPTION_KEY`)
- Dashboard: repos tracked, commits today, open PRs
- `GithubDailySummaryScheduler` runs daily at **10:00**

### External Completions API

- `OpenAiServiceImpl` calls OpenAI chat completions
- Used by: chat, email summaries, behavior learning, compose, GitHub/calendar summaries
- User can select preferred model via `/api/ai/models`

---

## Background Jobs (Schedulers)

| Scheduler | Schedule | Action |
|-----------|----------|--------|
| `GmailSyncScheduler` | Every 5 min | Sync inbox for connected users |
| `DailyEmailSummaryScheduler` | Daily 10:00 | Generate email daily summaries |
| `GithubDailySummaryScheduler` | Daily 10:00 | Generate GitHub activity summaries |

---

## Chat & Intent Routing

The chat module accepts natural language and routes to the appropriate backend capability:

- Email search / summarize → `email` services
- Calendar queries → `calendar` services
- GitHub activity → `github` services
- General conversation → OpenAI completion

Conversations and messages are persisted (`conversation`, `message` tables) for history and workspace UI.

---

## Subscription & Quotas

| Plan | Drafts/day | Auto-draft | Backfill |
|------|------------|------------|----------|
| Free | 5 | No | No |
| Edge | 50 | Yes | Yes |
| Pinnacle | Unlimited | Yes | Yes |

Enforced by `BehaviorIntelligenceQuotaService`. Razorpay handles payment; subscription state in `subscription` table.

---

## Frontend Integration

| Concern | Implementation |
|---------|----------------|
| Base URL | `REACT_APP_BACKEND_URL` (e.g. `https://api.abhiram.tech/api`) |
| Token storage | `localStorage.token` |
| 401 handling | Clear token, redirect to `/login` |
| Google login | Redirect to `{backend}/oauth2/authorization/google` |
| OAuth callback | `/oauth-success?token=...` on frontend |

---

## Security Summary

| Area | Approach |
|------|----------|
| API auth | JWT (HS256), 24h expiry |
| Passwords | BCrypt |
| GitHub tokens | AES encryption in DB |
| CORS | Whitelist: localhost, synoptiq.abhiram.tech, Vercel preview |
| Public routes | `/api/auth/**`, `/oauth2/**`, GitHub callback, Swagger, actuator health |

---

## Future / Stub Modules

- `workflow/` — placeholder for workflow engine
- `integration/pending/` — stub responses for integrations not yet built (Slack, Notion API, etc.)

These exist for frontend routing and chat intent placeholders but are not production integrations yet.
