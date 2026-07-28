# Deployment (Overview)

High-level production architecture for Synoptiq. **No credentials, secrets, or server access details are included in this public documentation.**

---

## Production URLs

| Service | URL |
|---------|-----|
| Frontend | https://synoptiq.abhiram.tech |
| API | https://api.abhiram.tech |
| API docs (Swagger) | https://api.abhiram.tech/swagger-ui/index.html |

---

## Infrastructure Diagram

```
                    ┌─────────────────┐
   Users ──────────►│  HTTPS / CDN    │
                    │  (reverse proxy)│
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼                             ▼
    ┌─────────────────┐           ┌─────────────────┐
    │  Static Frontend │           │  Docker Container│
    │  (React build)   │           │  Spring Boot API │
    └─────────────────┘           └────────┬────────┘
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
- **Container:** Docker image built from multi-stage Dockerfile
- **Host:** AWS EC2 instance
- **Process:** `java -jar synoptiq.jar` on port 8080 behind reverse proxy

### Frontend
- **Build:** `yarn build` → static files in `build/`
- **Serve:** Static hosting (Nginx or equivalent) at `synoptiq.abhiram.tech`

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

These are configured on the server — **never committed to git:**

| Variable | Purpose |
|----------|---------|
| `DB_URL`, `DB_USERNAME`, `DB_PASSWORD` | PostgreSQL connection |
| `JWT_SECRET` | JWT signing key |
| `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET` | Google OAuth |
| `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET` | GitHub OAuth |
| `GITHUB_OAUTH_REDIRECT_URI` | GitHub callback URL |
| `OPENAI_API_KEY` | Completions API |
| `FRONTEND_URL` | OAuth redirect target |
| `APP_ENCRYPTION_KEY` | Encrypt GitHub tokens at rest |
| `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET` | Payment processing |
| `DDL_AUTO` | Hibernate schema mode (`validate` in production) |

---

## Deployment Flow (summary)

1. Pull latest code on server (private repo)
2. Build Docker image: `./mvnw clean package -DskipTests`
3. Run database migrations if schema changed
4. `docker compose up -d --build`
5. Verify health: `GET /actuator/health`
6. Frontend: rebuild static assets and deploy to static host

---

## Security Practices

- All secrets in environment variables, not source code
- `DDL_AUTO=validate` in production
- GitHub OAuth tokens encrypted before database storage
- JWT with 24-hour expiry
- CORS restricted to known frontend origins
- HTTPS enforced on all public endpoints

---

## Health Monitoring

- Spring Actuator: `/actuator/health`
- Scheduled jobs: Gmail sync (5 min), daily summaries (10:00 AM)

---

For detailed deployment procedures, refer to private repository documentation or contact the author.
