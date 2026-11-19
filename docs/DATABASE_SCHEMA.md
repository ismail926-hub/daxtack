# Database Schema (Supabase Postgres)
Status: PROVISIONAL design for Phase 2 — not approved, not implemented. Storage mode, size caps and retention (`expires_at`, 500 KB cap) are deferred decisions.
All tables: RLS enabled; policies restrict to `user_id = auth.uid()`. Migrations are additive and committed in `backend/migrations/`. No destructive migration or production change without owner approval.
## profiles
id uuid PK (= auth.users.id) · display_name text · plan text default 'free' · created_at timestamptz
## analyses
id uuid PK · user_id uuid FK · title text · source text default 'github_actions' · redacted_log text (<=500 KB) · log_sha256 text · error_category text · confidence numeric · result jsonb · redaction_count int · created_at timestamptz · expires_at timestamptz
Index (user_id, created_at desc)
## usage_counters
user_id uuid · period date · analyses_count int · PK(user_id, period)
Raw logs are never stored.
