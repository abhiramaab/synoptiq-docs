# Frontend Architecture

Overview of the Synoptiq React application's structure, routing, state management, and API layer.

> Frontend source code is in a **private repository**. This document describes the architecture only.

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
| `/privacy`, `/terms`, `/contact` | Legal pages |

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
baseURL: process.env.REACT_APP_BACKEND_URL  // e.g. https://api.abhiram.tech/api
```

**Request interceptor:** attaches `Authorization: Bearer ${localStorage.token}`

**Response interceptor:** on `401` → clear token, redirect to `/login`

### Service modules

| Module | Backend paths |
|--------|---------------|
| `gmailService.js` | `/gmail/dashboard`, `/emails/stats` |
| `githubService.js` | `/github/*` |
| `calendarService.js` | `/calendar/*` |
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

### Workspace
- Conversational AI chat with conversation history sidebar

### Settings
- Profile, connected apps, usage quotas, Razorpay billing

### Daily Brief
- Aggregated inbox, calendar, and GitHub summary for the day

---

## Authentication Flow

```
LoginCard
  → redirect to BACKEND/oauth2/authorization/google
  → Google consent
  → Backend issues JWT
  → Redirect: FRONTEND/oauth-success?token=JWT
  → Token saved to localStorage
  → Navigate /dashboard
```

See [AUTHENTICATION.md](./AUTHENTICATION.md) for full backend auth details.

---

## Environment Variables

| Variable | Example |
|----------|---------|
| `REACT_APP_BACKEND_URL` | `https://api.abhiram.tech/api` |
| `REACT_APP_BACKEND_BASE_URL` | `https://api.abhiram.tech` |

---

## Related Documentation

- [Architecture](./ARCHITECTURE.md)
- [API Reference](./API.md)
- [Authentication](./AUTHENTICATION.md)
