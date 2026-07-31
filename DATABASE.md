# Database

Synoptiq uses **PostgreSQL** with **Spring Data JPA** and Hibernate. Schema is managed via `spring.jpa.hibernate.ddl-auto` (default `update` in dev; use `validate` in production after migrations).

Production database: **Neon PostgreSQL** (serverless, connection pooling).

---

## Entity Relationship Overview

```
users ──┬── gmail_tokens (1:1)
        ├── emails (1:N) ─── email_attachments (1:N)
        ├── notifications (1:N)
        ├── watched_threads (1:N)
        ├── github_accounts (1:1) ─── github_repos (1:N)
        ├── github_daily_summaries (1:N)
        ├── daily_email_summaries (1:N)
        ├── conversation (1:N) ─── messages (1:N)
        ├── agent_session_states (1:1 per conversation)
        ├── semantic_memories (1:N)
        ├── compose_drafts (1:N) ─── compose_attachments (1:N)
        ├── reply_drafts (1:N) → emails
        ├── user_behavior_profiles (1:1)
        ├── user_behavior_settings (1:1)
        ├── behavior_samples (1:N)
        ├── subscription (1:1)
        ├── saved_searches (1:N)
        └── search_history (1:N)
```

All user-owned data is scoped by `user_id` foreign key.

---

## Core Tables

### `users`

| Column | Type | Notes |
|--------|------|-------|
| id | BIGINT PK | |
| email | VARCHAR UNIQUE | Login identifier |
| password | VARCHAR | BCrypt (local users) |
| username | VARCHAR | Display name |
| google_id | VARCHAR | Google OIDC subject |
| provider | ENUM | LOCAL, GOOGLE |
| role | ENUM | USER, ADMIN |
| preferred_ai_model | VARCHAR | e.g. gpt-4o-mini |
| profile_picture | VARCHAR | URL from Google |

### `gmail_tokens`

| Column | Notes |
|--------|-------|
| user_id | FK → users (1:1) |
| access_token | Google OAuth access token |
| refresh_token | For token refresh |
| expires_at | Token expiry timestamp |

### `emails`

| Column | Notes |
|--------|-------|
| gmail_id | Unique Gmail message ID |
| thread_id | Gmail thread |
| sender, recipient | Email addresses |
| subject, body, snippet | Content |
| received_at | Message timestamp |
| is_read, is_summarized | Flags |
| user_id | FK → users |

### `email_attachments`

| Column | Notes |
|--------|-------|
| email_id | FK → emails |
| attachment_id | Gmail attachment ID |
| filename, mime_type, size_bytes | Metadata |

### `github_accounts`

| Column | Notes |
|--------|-------|
| user_id | FK → users (1:1) |
| github_username | |
| access_token | **Encrypted** at rest |

### `reply_drafts`

| Column | Notes |
|--------|-------|
| email_id | FK → emails |
| status | PENDING, APPROVED, REJECTED, etc. |
| draft_body | AI-generated reply text |
| resolved_at | When user acted |

### `subscription`

| Column | Notes |
|--------|-------|
| user_id | FK → users |
| razorpay_subscription_id | |
| plan_type | FREE, EDGE, PINNACLE |
| billing_cycle | MONTHLY, ANNUAL |
| status | ACTIVE, CANCELLED, etc. |

---

## Search Tables

### `saved_searches`

Stores named queries from Crawls / Advanced Search.

### `search_history`

Audit trail of executed searches with result counts.

---

## Chat Tables

### `conversation`

| Column | Notes |
|--------|-------|
| title | Conversation name |
| pinned | Boolean |
| user_id | FK → users |

### `message`

Chat messages linked to conversations (role: user/assistant, content, timestamps).

### `agent_session_states`

Per-conversation agent context for multi-turn tool use (agent platform).

| Column | Notes |
|--------|-------|
| conversation_id | FK → conversation (unique) |
| tool_context | Serialized tool state |
| domains | Active integration domains |
| updated_at | Last update timestamp |

### `semantic_memories`

Long-term user memories for the agent platform.

| Column | Notes |
|--------|-------|
| user_id | FK → users |
| content | Memory text |
| embedding_json | Optional embedding vector (JSON) |
| memory_type | Category label |
| created_at | Timestamp |

---

## Behavior Tables

### `user_behavior_profiles`

Learned writing style: tone, formality, length preference, emoji usage, sample count.

### `user_behavior_settings`

Toggles for auto-draft, backfill, notifications.

### `behavior_samples`

Individual sent-email samples used for learning.

---

## Migrations

Production schema scripts (run in private backend repo):

```
scripts/fix-production-schema.sql
scripts/migrate-agent-platform.sql   ← agent_session_states, semantic_memories
```

Run against Neon **before** deploying with `DDL_AUTO=validate`:

```bash
psql "$DB_URL" -f scripts/migrate-agent-platform.sql
psql "$DB_URL" -f scripts/fix-production-schema.sql
```

Then restart backend:

```bash
docker compose down && docker compose up -d --build
```

---

## Connection Configuration

```properties
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
spring.jpa.hibernate.ddl-auto=${DDL_AUTO:update}
```

**Neon tip:** Use `prepareThreshold=0` in JDBC URL for PgBouncer compatibility (configured in `application.properties`).

---

## Indexing Recommendations

For production at scale, consider indexes on:

- `emails(user_id, received_at DESC)`
- `emails(gmail_id)` UNIQUE
- `notifications(user_id, is_read)`
- `reply_drafts(user_id, status)`
- `search_history(user_id, created_at DESC)`

Hibernate `ddl-auto=update` does not always create optimal indexes — add via migration scripts as traffic grows.
