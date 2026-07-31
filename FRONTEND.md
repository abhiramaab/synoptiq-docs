# Frontend Architecture

Overview of the Synoptiq React application's structure, routing, state management, and API layer.

> Frontend source code is in a **private repository**. This document describes the architecture only.  
> **Production URL:** [usesynoptiq.com](https://usesynoptiq.com) (hosted on Vercel)

---

## Tech Stack

| Category | Technology |
|----------|------------|
| Framework | React 18.3 |
| Build | Create React App + CRACO |
| Routing | React Router 7 |
| Data fetching | TanStack Query, Axios |
| Styling | Tailwind CSS 3, shadcn/ui, Framer Motion |
| Forms | React Hook Form + Zod |
| Payments | Razorpay checkout |
| Hosting | Vercel (auto-deploy from GitHub `main`) |

---

## Application Bootstrap

```
src/index.js
  └── App.js (BrowserRouter + AuthProvider + Routes)
        └── ProtectedRoute → AppShell (Sidebar + TopNav + Outlet)
```

| File | Role |
|------|------|
| `src/index.js` | ReactDOM render, global CSS |
| `src/App.js` | Route definitions, lazy-loaded pages |
| `src/context/AuthContext.jsx` | User session, `GET /users/me` |
| `src/components/ProtectedRoute.jsx` | JWT gate |
| `src/components/layout/AppShell.jsx` | App chrome |

---

## Routing

### Public routes

| Path | Page |
|------|------|
| `/` | LandingPage |
| `/login` | Login |
| `/oauth-success` | OAuthSuccess |
| `/privacy`, `/terms`, `/contact` | Legal pages (updated July 31, 2026) |

### Protected routes (require `localStorage.token`)

| Path | Page | Notes |
|------|------|-------|
| `/dashboard` | Dashboard (Crawls) | Home after login |
| `/brief` | DailyBrief | Lazy |
| `/automations/:slug` | AutomationPage | gmail, github, google-calendar, notion |
| `/workspace` | Workspace | AI chat |
| `/settings` | Settings | Profile, billing |
| `/notifications` | Notifications | |
| `/integrations` | Integrations | |
| `/search` | → redirect `/dashboard` | |
| `/profile` | → redirect `/settings` | |

---

## Navigation

Sidebar sections:

```
Primary:     Daily Brief, Crawls
Automations: Gmail, GitHub, Google Calendar, Notion
Workspace:   Workspace, Notifications, History, Analytics, Integrations
Account:     Settings, Help
```

---

## API Layer

### Axios client

```javascript
baseURL: process.env.REACT_APP_BACKEND_URL  // https://api.abhiram.tech/api
```

**Request interceptor:** attaches `Authorization: Bearer ${localStorage.token}`

**Response interceptor:** on `401` → clear token, redirect to `/login`

### Service modules

| Module | Backend paths |
|--------|---------------|
| `gmailService.js` | `/gmail/dashboard`, `/emails/stats` |
| `githubService.js` | `/github/*` |
| `calendarService.js` | `/calendar/status`, `/calendar/events/upcoming`, `/calendar/events/today`, `/calendar/summary/today` |
| `chatService.js` | `/chat/*` |
| `searchService.js` | `/search/*` |
| `behaviorService.js` | `/behavior/*`, `/drafts/*` |
| `composeService.js` | `/compose/*` |
| `notificationService.js` | `/notification` |
| `dashboardService.js` | Aggregates multiple APIs for overview |
| `subscriptionService.js` | `/subscription/*` |

---

## State Management

| Pattern | Used for |
|---------|----------|
| **React Context** | Auth (`AuthContext`) |
| **TanStack Query** | Server state (dashboard, search, drafts, notifications) |
| **Local useState** | Form inputs, UI toggles |
| **localStorage** | JWT token (`token` key) |

---

## Key Pages

### Dashboard (Crawls)
- Greeting, quick actions, unified search panel, overview stats grid, activity timeline
- Attachment downloads with Gmail-first path and clear error messages

### Automations (`/automations/:slug`)
Per-integration dashboards for Gmail, GitHub, Google Calendar, and Notion:

- Shows **live connection status** from the backend
- Hides "Connect" toolbar actions when the integration is already linked
- Displays real data (not static placeholders) for stats and upcoming events

**GitHub:** `githubToolbarActions` computed from `githubService.getDashboard()` — Connect only when disconnected.

**Calendar:** Fetches `calendarService.getStatus()`, `getUpcomingEvents(14)`, and `getTodaySummary()` on load.

**Gmail:** Connect button hidden when Google account is linked via OAuth.

### Workspace
- Conversational AI chat with conversation history sidebar
- Sends messages via `POST /chat` for reliable responses
- Model selector shows OpenAI models only
- Natural-language queries across Gmail, GitHub, and Calendar via backend intent routing

### Settings
- Profile, connected apps, usage quotas, Razorpay billing

### Daily Brief
- Aggregated inbox, calendar, and GitHub summary for the day

### Legal pages
- `Privacy.jsx` and `Terms.jsx` updated July 31, 2026 with Google API data usage, account terms, and `usesynoptiq.com` references

---

## Authentication Flow

```
LoginCard
  → window.location = https://api.abhiram.tech/oauth2/authorization/google
  → Google consent
  → Backend issues JWT
  → Redirect: https://usesynoptiq.com/oauth-success?token=JWT
  → Token saved to localStorage
  → Navigate /brief
```

> OAuth always starts on the **API domain** (`api.abhiram.tech`), not the frontend domain. The backend `FRONTEND_URL` env var controls the post-login redirect.

See [AUTHENTICATION.md](./AUTHENTICATION.md) for full backend auth details.

---

## Environment Variables

| Variable | Production value |
|----------|------------------|
| `REACT_APP_BACKEND_URL` | `https://api.abhiram.tech/api` |
| `REACT_APP_BACKEND_BASE_URL` | `https://api.abhiram.tech` |

Set in Vercel project → Settings → Environment Variables.

---

## Vercel Deployment

- **Repo:** `github.com/abhiramaab/synoptiq-frontend`
- **Branch:** `main` (auto-deploy on push)
- **Domains:** `usesynoptiq.com`, `www.usesynoptiq.com`
- **Build:** `npm run build` (CRA output in `build/`)

After a `git push --force` that rewrites history, trigger a fresh deploy with a new commit (Vercel cannot redeploy deleted commit SHAs).

---

## Related Documentation

- [Architecture](./ARCHITECTURE.md)
- [API Reference](./API.md)
- [Authentication](./AUTHENTICATION.md)
- [Deployment](./DEPLOYMENT.md)
