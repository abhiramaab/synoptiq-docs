# Deployment (Overview)

High-level production architecture for Synoptiq. **No credentials, secrets, or server access details are included in this public documentation.**

---

## Production URLs

| Service | URL | Hosting |
|---------|-----|---------|
| **Frontend** | https://usesynoptiq.com | Vercel |
| **Frontend (www)** | https://www.usesynoptiq.com | Vercel |
| **API** | https://api.abhiram.tech | AWS EC2 + Docker |
| **API docs (Swagger)** | https://api.abhiram.tech/swagger-ui/index.html | EC2 |

---

## Domain Architecture

```
usesynoptiq.com  ──►  Vercel (React static build)
www.usesynoptiq.com ──►  Vercel

api.abhiram.tech ──►  EC2 (Spring Boot API, port 8080 behind Nginx)
```

> **Critical:** `usesynoptiq.com` must point to **Vercel**, not the EC2 API server.  
> If the frontend domain points at the API IP, users get SSL errors (`api.abhiram.tech` certificate on `usesynoptiq.com`) and OAuth `redirect_uri_mismatch`.

### DNS (Hostinger → Vercel)

| Type | Name | Value |
|------|------|-------|
| CNAME or A | `@` | Vercel-provided records (from Vercel → Domains) |
| CNAME | `www` | `cname.vercel-dns.com` (or Vercel alias) |

Do **not** add an A record for `usesynoptiq.com` pointing at the EC2 API IP (`13.200.154.148`). That IP is only for `api.abhiram.tech`.

---

## Infrastructure Diagram

```
                    ┌─────────────────┐
   Users ──────────►│  Vercel CDN     │  usesynoptiq.com
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼                             ▼
    ┌─────────────────┐           ┌─────────────────┐
    │  React Frontend  │           │  Nginx + Docker │
    │  (Vercel build)  │  REST/JWT │  Spring Boot API │
    └─────────────────┘           │  api.abhiram.tech│
                                    └────────┬────────┘
                                             │
                                             ▼
                                    ┌─────────────────┐
                                    │  PostgreSQL     │
                                    │  (Neon cloud)   │
                                    └─────────────────┘
```

---

## Components

### Backend
- **Runtime:** Java 21, Spring Boot 3.5
- **Container:** Docker image built from Dockerfile
- **Host:** AWS EC2 instance
- **Process:** `java -jar` on port 8080 behind Nginx reverse proxy

### Frontend
- **Build:** `npm run build` → static files in `build/`
- **Host:** Vercel (auto-deploy from GitHub `main`)
- **Custom domain:** `usesynoptiq.com`

### Database
- **Engine:** PostgreSQL
- **Hosting:** Neon (serverless, connection pooling)
- **Schema:** Managed via Hibernate + SQL migration scripts

### External services
- Google Cloud (OAuth, Gmail API, Calendar API)
- GitHub (OAuth + REST API)
- OpenAI (completions API)
- Razorpay (subscriptions)

---

## Environment Variables (names only)

### Backend (EC2 / Docker)

| Variable | Production example |
|----------|-------------------|
| `DB_URL`, `DB_USERNAME`, `DB_PASSWORD` | Neon PostgreSQL connection |
| `JWT_SECRET` | JWT signing key |
| `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET` | Google OAuth |
| `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET` | GitHub OAuth |
| `GITHUB_OAUTH_REDIRECT_URI` | `https://api.abhiram.tech/api/github/oauth/callback` |
| `OPENAI_API_KEY` | Completions API |
| **`FRONTEND_URL`** | **`https://usesynoptiq.com`** |
| `APP_ENCRYPTION_KEY` | Encrypt GitHub tokens at rest |
| `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET` | Payment processing |
| `DDL_AUTO` | `validate` in production |

### Frontend (Vercel)

| Variable | Production value |
|----------|------------------|
| `REACT_APP_BACKEND_URL` | `https://api.abhiram.tech/api` |
| `REACT_APP_BACKEND_BASE_URL` | `https://api.abhiram.tech` |

---

## Deployment Flow

### Backend (EC2)

```bash
cd ~/synoptiq
git pull origin main
# Run migrations if schema changed (see DATABASE.md)
docker compose up -d --build
```

### Frontend (Vercel)

Push to `main` on `github.com/abhiramaab/synoptiq-frontend` → Vercel auto-builds and deploys.

Manual redeploy: Vercel dashboard → Deployments → Redeploy (must be a commit that still exists on `main`).

### Database migrations

Before restarting backend with new agent tables:

```bash
psql "$DB_URL" -f scripts/migrate-agent-platform.sql
```

---

## Security Practices

- All secrets in environment variables, not source code
- `DDL_AUTO=validate` in production
- GitHub OAuth tokens encrypted before database storage
- JWT with 24-hour expiry
- CORS restricted to `usesynoptiq.com`, `synoptiq.abhiram.tech`, Vercel previews
- HTTPS enforced on all public endpoints
- Frontend and API on separate domains with correct SSL certificates

---

## Health Monitoring

- Spring Actuator: `/actuator/health`
- Scheduled jobs: Gmail sync (5 min), daily summaries (10:00 AM)
- Vercel: deployment status and build logs in dashboard

---

For detailed deployment procedures, refer to private repository documentation or contact the author.
