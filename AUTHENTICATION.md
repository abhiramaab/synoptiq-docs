# Authentication

Synoptiq uses **JWT bearer tokens** for API authentication, with **Google OAuth2** as the primary sign-in method and a separate **GitHub OAuth** flow for linking developer accounts.

---

## Overview

| Flow | Use case | Token issued |
|------|----------|--------------|
| Email register/login | Alternative sign-in | JWT in JSON body |
| Google OAuth | Primary sign-in (+ Gmail/Calendar access) | JWT via frontend redirect |
| GitHub OAuth | Link GitHub to existing account | No new JWT; stores GitHub token |

All protected `/api/**` routes require:

```
Authorization: Bearer <jwt>
```

---

## 1. Google OAuth Login (Primary)

### Sequence

```
┌────────┐          ┌─────────────┐          ┌────────┐          ┌──────────┐
│ Browser│          │  Frontend   │          │ Backend│          │  Google  │
└───┬────┘          └──────┬──────┘          └───┬────┘          └────┬─────┘
    │  Click "Sign in"     │                     │                    │
    │─────────────────────►│                     │                    │
    │                      │  Redirect to        │                    │
    │                      │  /oauth2/authorization/google              │
    │──────────────────────────────────────────►│                    │
    │                      │                     │  OAuth consent     │
    │─────────────────────────────────────────────────────────────────►│
    │                      │                     │◄───────────────────│
    │                      │                     │  /login/oauth2/code/google
    │                      │                     │  Create/update User
    │                      │                     │  Save gmail_tokens
    │                      │                     │  Issue JWT
    │  Redirect https://usesynoptiq.com/oauth-success?token=JWT          │
    │◄─────────────────────────────────────────│                    │
    │  Save token to localStorage              │                    │
    │  Redirect /dashboard                     │                    │
```

### Google Scopes Requested

- `openid`, `email`, `profile`
- `https://www.googleapis.com/auth/gmail.readonly`
- `https://www.googleapis.com/auth/gmail.send`
- `https://www.googleapis.com/auth/calendar.readonly`

### Key backend classes

| Class | Role |
|-------|------|
| `OAuth2AuthorizationRequestResolverConfig` | Adds `access_type=offline`, `prompt=consent` |
| `CustomOidcUserService` | Creates/updates `User` from Google profile |
| `OAuth2SuccessHandler` | Saves Gmail tokens, issues JWT, redirects to frontend |
| `GoogleOAuthTokenService` | Persists tokens to `gmail_tokens` |

### Frontend handling

1. `LoginCard.jsx` redirects to `{BACKEND_ROOT}/oauth2/authorization/google`
2. `OAuthSuccess.jsx` reads `?token=` from URL, stores in `localStorage`, navigates to `/dashboard`

---

## 2. Email / Password Auth

### Register

```http
POST /api/auth/register
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securePassword",
  "username": "Abhirama"
}
```

### Login

```http
POST /api/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securePassword"
}
```

### Response (both)

```json
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "email": "user@example.com",
  "username": "Abhirama"
}
```

Passwords are hashed with **BCrypt** before storage.

---

## 3. JWT Validation (Every API Request)

```
Request with Authorization: Bearer <token>
        │
        ▼
JwtAuthenticationFilter
        │
        ├─ Parse & verify signature (HS256, JWT_SECRET)
        ├─ Extract email from subject claim
        ├─ Load User from database
        └─ Set SecurityContext authentication
        │
        ▼
Controller (user is authenticated)
```

| Setting | Value |
|---------|-------|
| Algorithm | HS256 |
| Subject | User email |
| Expiry | 24 hours (`jwt.expiration=86400000`) |
| Secret | `JWT_SECRET` env var |

### Frontend interceptor (`src/lib/api.js`)

- Attaches `Authorization` header from `localStorage.token`
- On `401`: clears token, redirects to `/login`

---

## 4. GitHub OAuth (Account Linking)

GitHub OAuth is **not** a login method — it links GitHub to an already-authenticated user.

### Sequence

```
1. User logged in (has JWT)
2. GET /api/github/oauth/authorize-url  → returns GitHub authorize URL with signed state
3. User approves on GitHub
4. GET /api/github/oauth/callback?code=...&state=...  (public)
5. Backend exchanges code, encrypts token, saves to github_accounts
6. Redirect to FRONTEND_URL/automations/github?github=connected
```

### Security

- Callback is public (browser redirect) but uses **signed state** to prevent CSRF
- GitHub access token encrypted with `APP_ENCRYPTION_KEY` before DB storage
- Disconnect: `POST /api/github/oauth/disconnect`

---

## 5. Security Configuration

### Public routes (no JWT)

- `/api/auth/**`
- `/oauth2/**`, `/login/**`
- `/api/github/oauth/callback`
- `/swagger-ui/**`, `/v3/api-docs/**`
- `/actuator/**`

### Protected routes

- All other `/api/**` endpoints

### CORS allowed origins

- `http://localhost:5173`, `http://localhost:3000`
- `https://usesynoptiq.com`, `https://www.usesynoptiq.com`
- `https://synoptiq.abhiram.tech`, `https://www.synoptiq.abhiram.tech`
- `https://synoptiq.vercel.app`

Configured in `SecurityConfig.java`. Production `FRONTEND_URL` on the backend should be `https://usesynoptiq.com`.

---

## 6. User Entity & Providers

| Field | Description |
|-------|-------------|
| `email` | Unique identifier |
| `password` | BCrypt hash (local users only) |
| `googleId` | Google subject ID |
| `provider` | `LOCAL` or `GOOGLE` |
| `role` | `USER` or `ADMIN` |
| `preferredAiModel` | Selected completion model |

---

## Troubleshooting

| Issue | Likely cause |
|-------|--------------|
| 401 on all requests | Expired or missing JWT |
| Google login redirects but no Gmail sync | User denied scopes; re-login with consent |
| Calendar "not connected" | `calendar.readonly` scope missing; re-authorize Google |
| GitHub callback error | `GITHUB_OAUTH_REDIRECT_URI` mismatch with GitHub app settings |
| SSL error on `usesynoptiq.com` | Frontend domain pointing at API server IP instead of Vercel |
| OAuth `redirect_uri_mismatch` | Login started on wrong host; must use `api.abhiram.tech/oauth2/authorization/google` |
