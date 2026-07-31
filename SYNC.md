# Keeping Docs in Sync

When you update documentation in your **private backend repo** (`synoptiq-main/docs/`), copy matching sections to this public repo:

```bash
# From your machine
cp ~/Downloads/synoptiq-main/docs/ARCHITECTURE.md ~/Downloads/synoptiq-docs/
cp ~/Downloads/synoptiq-main/docs/API.md ~/Downloads/synoptiq-docs/
cp ~/Downloads/synoptiq-main/docs/AUTHENTICATION.md ~/Downloads/synoptiq-docs/
cp ~/Downloads/synoptiq-main/docs/DATABASE.md ~/Downloads/synoptiq-docs/

cd ~/Downloads/synoptiq-docs
git add -A
git commit -m "docs: sync from private backend repo"
git push
```

**Files unique to this public repo** (edit here directly):
- `README.md` — landing page, links, feature list, recent updates
- `OVERVIEW.md` — recruiter-friendly summary
- `DEPLOYMENT.md` — production overview, domain/DNS setup (no secrets)
- `FRONTEND.md` — React app architecture, Vercel, automations UI
- `SYNC.md` — this file

**Sections that change frequently** (update in both repos when shipping features):
- Chat / agent platform (`ARCHITECTURE.md`)
- New REST endpoints (`API.md`)
- New database tables (`DATABASE.md`)
- Domain or deployment changes (`DEPLOYMENT.md`, `FRONTEND.md`)

**Never copy to public repo:**
- `.env`, `application.properties` with real values
- SQL dumps, migration scripts with real data
- Source code
