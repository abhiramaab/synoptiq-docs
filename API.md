# API Reference

Base URL: `https://api.abhiram.tech/api` (production) or `http://localhost:8080/api` (local)

**Authentication:** Unless noted *Public*, send:

```
Authorization: Bearer <jwt_token>
```

**Interactive docs:** [Swagger UI](https://api.abhiram.tech/swagger-ui/index.html) (when deployed)

---

## Auth

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/auth/register` | Public | Register with email/password |
| POST | `/auth/login` | Public | Login, returns JWT |

---

## Users

| Method | Path | Description |
|--------|------|-------------|
| GET | `/users/me` | Current user profile |
| PATCH | `/users/me` | Update profile fields |

---

## Gmail & Emails

| Method | Path | Description |
|--------|------|-------------|
| POST | `/emails/sync` | Trigger manual inbox sync |
| GET | `/emails` | List emails (paginated) |
| GET | `/emails/{id}` | Single email detail |
| GET | `/emails/search` | Keyword search |
| GET | `/emails/search/natural` | Natural-language Gmail search |
| GET | `/emails/stats` | Email statistics |
| POST | `/emails/{id}/summarize` | Summarize one email |
| POST | `/emails/summarize` | Batch summarize |
| GET | `/emails/summarize/date-range` | Summaries by date range |
| GET | `/emails/daily-summary/today` | Today's inbox summary |
| GET | `/emails/daily-summary/history` | Past daily summaries |
| GET | `/emails/attachments/{attachmentId}/download` | Download attachment |
| GET | `/gmail/dashboard` | Gmail dashboard metrics |
| GET | `/gmail/emails` | Gmail-native email list |

---

## Calendar

| Method | Path | Description |
|--------|------|-------------|
| GET | `/calendar/status` | Connection status + `upcomingToday`, `meetingsThisWeek`, `upcomingNext7Days` |
| GET | `/calendar/events/today` | Today's calendar events |
| GET | `/calendar/events/upcoming?days=14` | Upcoming events (1–30 day window, default 14) |
| GET | `/calendar/search?q=` | Search events by query |
| GET | `/calendar/summary/today` | AI summary of today's meetings |
| POST | `/calendar/events/create` | Create event from natural-language prompt (`{ "prompt": "..." }`) |

---

## GitHub

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/github/oauth/authorize-url` | JWT | Start GitHub OAuth |
| GET | `/github/oauth/callback` | Public | OAuth callback (browser) |
| POST | `/github/oauth/disconnect` | JWT | Remove GitHub connection |
| GET | `/github/dashboard` | JWT | Repos, commits, PRs |
| GET | `/github/repos` | JWT | Tracked repositories |
| POST | `/github/resync` | JWT | Re-sync GitHub data |
| GET | `/github/summary/today` | JWT | Today's dev summary |
| GET | `/github/summary/history` | JWT | Past summaries |
| GET | `/github/issues` | JWT | Issues (filtered) |
| GET | `/github/workflows` | JWT | Workflow runs |
| GET | `/github/releases` | JWT | Releases |
| GET | `/github/search` | JWT | Search commits/PRs |

---

## Chat (AI Workspace)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/chat` | Send message, get AI response (primary path used by frontend) |
| POST | `/chat/stream` | Same as `/chat` but returns Server-Sent Events (`text/event-stream`) |
| GET | `/chat/history` | Message history (`?conversationId=` optional) |
| GET | `/chat/conversations` | List conversations (`?search=` optional) |
| POST | `/chat/conversations` | Create conversation |
| PATCH | `/chat/conversations/{id}` | Rename conversation |
| PATCH | `/chat/conversations/{id}/pin` | Pin/unpin |
| DELETE | `/chat/conversations/{id}` | Delete conversation |

**Chat request body:**

```json
{
  "message": "how many commits in the last 2 days?",
  "conversationId": 123,
  "model": "GPT_4_1_MINI"
}
```

The `model` field accepts internal enum names; the backend maps them to OpenAI API slugs via `AiModel.openAiApiId()`.

---

## Compose

| Method | Path | Description |
|--------|------|-------------|
| POST | `/compose/draft` | Create AI compose draft |
| GET | `/compose/{id}` | Get draft |
| PUT | `/compose/{id}` | Update draft |
| POST | `/compose/{id}/approve` | Approve and send |
| POST | `/compose/attachments` | Upload attachment (multipart) |
| GET | `/compose/attachments/pending` | Pending attachments |
| DELETE | `/compose/attachments/{attachmentId}` | Remove attachment |

---

## Reply Drafts (Behavior)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/drafts/pending` | Drafts awaiting approval |
| GET | `/drafts` | All drafts |
| GET | `/drafts/{id}` | Single draft |
| POST | `/drafts/{id}/approve` | Approve as-is |
| POST | `/drafts/{id}/approve-edited` | Approve with edits |
| POST | `/drafts/{id}/reject` | Reject draft |

---

## Behavior & Settings

| Method | Path | Description |
|--------|------|-------------|
| GET | `/behavior/profile` | Learned writing-style profile |
| POST | `/behavior/backfill` | Trigger learning from sent mail |
| GET | `/behavior/quota` | Plan quota usage |
| GET | `/behavior/settings` | Behavior toggles |
| PUT | `/behavior/settings` | Update toggles |

---

## Search (Crawls)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/search?q=&type=` | Unified search (email, github, calendar) |
| GET | `/search/saved` | Saved searches |
| POST | `/search/saved` | Save a search |
| DELETE | `/search/saved/{id}` | Delete saved search |
| GET | `/search/history` | Recent search history |

---

## Notifications

| Method | Path | Description |
|--------|------|-------------|
| GET | `/notification` | List notifications |
| POST | `/notification/{id}/read` | Mark as read |

---

## Watch Threads

| Method | Path | Description |
|--------|------|-------------|
| POST | `/watch/{emailId}` | Add email thread to watchlist |

---

## AI Models

| Method | Path | Description |
|--------|------|-------------|
| GET | `/ai/models` | Available models for user plan |
| PUT | `/ai/models/selected` | Set preferred model |

Only **OpenAI** models are active in the production UI. Internal enum values (e.g. `GPT_4_1_MINI`) are mapped to API slugs (`gpt-4.1-mini`) before calling OpenAI. Gemini and Grok appear as "coming soon".

---

## Subscriptions

| Method | Path | Description |
|--------|------|-------------|
| POST | `/subscription/create` | Create Razorpay subscription |
| POST | `/subscription/verify` | Verify payment signature |
| GET | `/subscription/status` | Current plan status |
| POST | `/subscription/cancel` | Cancel subscription |

---

## OAuth (Browser — not under `/api`)

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/oauth2/authorization/google` | Public | Start Google sign-in |
| GET | `/login/oauth2/code/google` | Public | Google OAuth callback |

---

## Health & Docs

| Path | Description |
|------|-------------|
| `/actuator/health` | Health check |
| `/swagger-ui/index.html` | Swagger UI |
| `/v3/api-docs` | OpenAPI JSON |

---

## Common Response Patterns

### Success
Most endpoints return `200 OK` with a JSON body. Create operations may return `201`.

### Errors
```json
{
  "timestamp": "2026-07-28T12:00:00",
  "status": 400,
  "error": "Bad Request",
  "message": "Human-readable error message"
}
```

### Pagination
List endpoints typically accept:
- `page` (0-based)
- `size`
- `sort` (e.g. `receivedAt,desc`)

---

## Rate Limits & Quotas

- Gmail sync: scheduled every 5 minutes per user
- Reply drafts: plan-dependent daily limits (see [ARCHITECTURE.md](./ARCHITECTURE.md))
- External API calls (Google, GitHub, OpenAI) subject to provider limits
