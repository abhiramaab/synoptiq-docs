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
│  ┌─────────────┐  ┌─────────────────────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │   auth/     │  │   chat/ + chat/agent/       │  │   email/    │  │   behavior/         │ │
│  │   user/     │  │   compose/                  │  │   search/   │  │   payment/          │ │
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
| `chat` | Conversations, messages, intent routing, agent orchestration |
| `chat.agent` | Router, planner, parallel tool executor, memory, response cache |
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
- Scopes: `calendar.readonly` + `calendar.events` (read and create)
- `CalendarService` lists events, supports natural-language date parsing
- `CalendarEventService` creates events from natural-language prompts
- `CalendarSummaryService` generates daily meeting summaries
- `GET /calendar/events/upcoming` powers automations page schedule view

### GitHub

- **Separate OAuth flow** from Google login
- Access token encrypted at rest (`APP_ENCRYPTION_KEY`)
- Dashboard: repos tracked, commits today, open PRs
- `GithubQueryParser` handles count queries and date ranges ("last 2 days")
- `GithubActivityFetcher` fetches repos in parallel with a 2-minute in-memory cache
- `GithubDailySummaryScheduler` runs daily at **10:00**

### External Completions API

- `OpenAiServiceImpl` calls OpenAI chat completions
- `AiModel.openAiApiId()` maps internal enum names (e.g. `GPT_4_1_MINI`) to API slugs (`gpt-4.1-mini`)
- Used by: chat, email summaries, behavior learning, compose, GitHub/calendar summaries
- User can select preferred model via `/api/ai/models` (OpenAI models active in production UI)

---

## Background Jobs (Schedulers)

| Scheduler | Schedule | Action |
|-----------|----------|--------|
| `GmailSyncScheduler` | Every 5 min | Sync inbox for connected users |
| `DailyEmailSummaryScheduler` | Daily 10:00 | Generate email daily summaries |
| `GithubDailySummaryScheduler` | Daily 10:00 | Generate GitHub activity summaries |

---

## Chat & Intent Routing

Workspace chat uses a **two-tier** architecture: structured intents bypass the agent for speed; everything else goes through the agent platform.

### Request flow

```
POST /api/chat
  │
  ▼
IntentServiceImpl.detectIntent()     ← keyword router; order matters
  │
  ├─ Structured intent (email, GitHub, calendar, workspace today)
  │     └─ Direct handler in ChatServiceImpl (no LLM when data suffices)
  │
  └─ NORMAL_CHAT
        ├─ FastChatResponder (instant greetings: "hi", "hello")
        └─ AgentOrchestrator
              ├─ AgentRouter → pick relevant tools
              ├─ ParallelToolExecutor → run tools concurrently
              ├─ AgentDirectResponseFormatter → return tool output directly (skip LLM)
              └─ Short LLM call only when narrative synthesis is needed
```

`POST /api/chat/stream` exposes the same logic via Server-Sent Events. The production frontend uses `POST /api/chat` for reliability.

### Intent types

| Intent | Example phrases | Handler |
|--------|-----------------|---------|
| `CALENDAR_CREATE` | "schedule a meeting tomorrow at 3pm", "add an event" | Create via Calendar API |
| `CALENDAR_SEARCH` | "do I have meetings tomorrow?" | List matching events |
| `CALENDAR_SUMMARY` | "what's on my calendar this week?" | Summary of schedule |
| `GITHUB_SEARCH` | "how many commits last 2 days?", "show recent PRs" | Parsed query + activity fetch |
| `GITHUB_SUMMARY` | "summarize my GitHub activity" | Dev summary |
| `EMAIL_SEARCH` | "find emails from John" | Gmail search |
| `EMAIL_COMPOSE` | "write an email to…" | Compose draft flow |
| `WORKSPACE_TODAY` | "what's on my plate today?" | Fast aggregated overview |
| `NORMAL_CHAT` | General questions | Agent orchestrator |

Calendar create detection runs **before** calendar search/summary so "add a meeting" does not get misrouted to a schedule listing.

### Agent platform (`chat/agent/`)

| Component | Role |
|-----------|------|
| `AgentRouter` | Selects which tools to invoke for a message |
| `AgentPlanner` | Builds execution plan from routed tools |
| `ParallelToolExecutor` | Runs Gmail, GitHub, Calendar, etc. tools concurrently |
| `AgentDirectResponseFormatter` | Formats structured tool results without LLM |
| `AgentCacheService` | Short-lived response cache for repeated queries |
| `ConversationMemoryService` | Per-conversation tool context (`agent_session_states`) |
| `SemanticMemoryService` | Long-term user memories (`semantic_memories`) |

### Agent tools

| Tool | Domain |
|------|--------|
| `GmailAgentTool` | Inbox search, summaries |
| `GitHubAgentTool` | Commits, PRs, repo activity |
| `CalendarAgentTool` | Events, schedule queries |
| `WorkspaceAgentTool` | Cross-domain day overview |
| `CrawlsAgentTool` | Unified search |
| `DailyBriefAgentTool` | Morning brief data |
| `NotificationAgentTool` | User notifications |
| `DriveAgentTool`, `KnowledgeAgentTool`, `AnalyticsAgentTool` | Stubs / partial |

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
| Google login | Redirect to `api.abhiram.tech/oauth2/authorization/google` |
| OAuth callback | `usesynoptiq.com/oauth-success?token=...` |

---

## Security Summary

| Area | Approach |
|------|----------|
| API auth | JWT (HS256), 24h expiry |
| Passwords | BCrypt |
| GitHub tokens | AES encryption in DB |
| CORS | Whitelist: localhost, usesynoptiq.com, synoptiq.abhiram.tech, Vercel preview |
| Public routes | `/api/auth/**`, `/oauth2/**`, GitHub callback, Swagger, actuator health |

---

## Future / Stub Modules

- `workflow/` — placeholder for workflow engine
- `integration/pending/` — stub responses for integrations not yet built (Slack, Notion API, etc.)

These exist for frontend routing and chat intent placeholders but are not production integrations yet.
