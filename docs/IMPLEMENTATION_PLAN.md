# Implementation Plan

## Phase 1 — Foundation + Analyzer Core (no external services; ~2–3 weeks part-time)
Why first: the analyzer is the product, needs no Supabase/hosting, and can be validated with users early.
Tasks:
1.1 Scaffold repo: folders, .gitignore, .editorconfig, .env.example, pre-commit (ruff, gitleaks), Dependabot.
1.2 Backend skeleton: pyproject, FastAPI app with `GET /health`, config via env (pydantic-settings), ruff/mypy/pytest configured.
1.3 CI (.github/workflows): lint, type-check, tests+coverage, bandit, pip-audit, gitleaks. Frontend job added in Phase 3.
1.4 Models + redaction: patterns for GitHub tokens (ghp_/gho_/ghs_/github_pat_), AWS keys, Slack/Stripe/Google/OpenAI/Anthropic-style keys, JWTs, Bearer/Authorization headers, URLs with credentials, private-key blocks, `password|secret|token|api_key=` assignments, high-entropy strings after secret-like keys. In-place per line; line count preserved. Hypothesis fuzz tests.
1.5 Parser: ANSI/timestamp strip with original line map, step/group segmentation, `##[error]`, exit codes.
1.6 Rule engine + ~15 initial rules across the categories in PRODUCT_REQUIREMENTS.md; ranking; explanations; fixes.
1.7 Fixture corpus: start with 20+ sanitized logs in tests/fixtures/logs with expected JSON (owner to supply real sanitized logs over time).
1.8 CLI: `python -m analyzer < log.txt` (JSON or readable output) for dogfooding.
1.9 Docs + DEVELOPMENT_PROGRESS update.
Exit criteria: CI green; analyzer coverage >= 85%; every redaction pattern tested; no raw log in stdout/stderr/log files; every finding has line numbers matching the original log; >=60% top-1 accuracy on fixtures; demo shown to a few developers.

## Phase 2 — API, DB, Auth (needs Supabase free project)
Migrations + RLS, JWT verification, analyses/usage endpoints, rate limiting, size limits, repository fakes, API tests incl. RLS isolation tests.
## Phase 3 — Frontend (Next.js, shadcn/ui): auth pages, analyzer page with evidence highlighting, history, dashboard, component + E2E tests.
## Phase 4 — Hardening + deployment (needs approval): security review, hosting choice, monitoring, privacy page, backups review.
## Phase 5 — Beta: first 10 users, feedback, rule improvements, pricing hypothesis.

## Environment variables
| Var | Where | Phase | Notes |
|---|---|---|---|
| ENVIRONMENT | backend | 1 | local/test/production |
| MAX_LOG_BYTES | backend | 1 | default 2097152 |
| AI_ENABLED | backend | 1 | default false |
| CORS_ORIGINS | backend | 2 | allow-list |
| SUPABASE_URL | backend + frontend | 2 | public |
| SUPABASE_ANON_KEY (publishable key) | backend + frontend | 2 | public by design; RLS protects data |
| SUPABASE_JWT_JWKS_URL or JWT secret | backend | 2 | depends on project's signing method (verify) |
| FREE_MONTHLY_LIMIT, RATE_LIMIT_PER_MIN | backend | 2 | |
| NEXT_PUBLIC_API_URL, NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY | frontend | 3 | |
| Service-role/secret key | NOT used by the API | — | only for owner-run admin/migrations, never in app or repo |
Real values live only in local `.env` files and hosting dashboards; only `.env.example` is committed.

## External services
GitHub + Actions (existing, free) · Supabase free project (Phase 2) · hosting TBD (Phase 4) · optional later: Sentry/uptime monitor, domain (paid, needs approval).

## Testing requirements
Unit (rules, redaction, parser) · property/fuzz (redaction never leaks seeded secrets, line count preserved) · golden fixtures (expected category + evidence lines) · API tests (auth, limits, errors, RLS isolation) · frontend component tests + Playwright smoke · CI blocks merge on failure · regression test added for every bug.

## Security requirements
Redact first; raw logs never stored/logged; body logging off; ReDoS-safe bounded regexes + size caps + per-request time budget; JWT verification; RLS on all tables; no service-role key in request path; CORS allow-list; security headers; dependency and secret scanning in CI; privacy notice that redaction is best-effort and users should not paste credentials intentionally; user data deletion.

## Decisions needing owner confirmation
1. Data access: user-JWT + RLS via Supabase client (recommended) vs direct SQL with explicit user filters.
2. Storage: redacted full log (500 KB cap, 90-day retention) vs only evidence excerpts (safer, less useful history). Recommended: redacted log + retention.
3. Phase ordering: analyzer core first (recommended) vs auth/UI first.
4. Open-source licence for the repo (none yet; choose later).
5. Python tooling: pip + pyproject (simple) vs uv (faster); recommended: uv if comfortable, else pip.
