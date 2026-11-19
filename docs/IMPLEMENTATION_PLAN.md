# Implementation Plan

## Phase 1 — Foundation + Analyzer Core (no external services; ~2–3 weeks part-time)
Why first: the analyzer is the product, needs no Supabase/hosting, and can be validated with users early.
Scope: IN — analyzer, CLI, tests, CI, tooling, local `GET /health`. OUT — database, auth, frontend, external services.
Tasks:
1.1 Safety + scaffold (first PR, before any code): .gitignore, .env.example (placeholders only), .editorconfig, pre-commit (ruff, gitleaks) with a gitleaks allowlist scoped to tests/fixtures/, Dependabot, folder skeleton.
1.2 Backend skeleton: uv project (pinned Python, committed uv.lock), pyproject, FastAPI app with `GET /health`, config via env (pydantic-settings), ruff/mypy/pytest configured.
1.3 CI (.github/workflows): lint, type-check, tests+coverage (85% gate for backend/analyzer/), bandit, pip-audit, gitleaks, report-only benchmark. Frontend job added in Phase 3.
1.4 Models + redaction: patterns for GitHub tokens (ghp_/gho_/ghs_/github_pat_), AWS keys, Slack/Stripe/Google/OpenAI/Anthropic-style keys, JWTs, Bearer/Authorization headers, URLs with credentials, private-key blocks, `password|secret|token|api_key=` assignments, high-entropy strings after secret-like keys. In-place per line; line count preserved. Hypothesis fuzz tests.
1.5 Parser: ANSI/timestamp strip with original line map, step/group segmentation, `##[error]`, exit codes.
1.6 Rule engine + ~15 initial rules across the categories in PRODUCT_REQUIREMENTS.md; ranking; explanations; fixes.
1.7 Fixture corpus (written BEFORE the rules in 1.6): >=20 synthetic fixtures with expected.json — 15 tuning, 5 held-out (a starting minimum, not statistically strong validation). Real logs only after owner review and sanitization; none committed without it.
1.8 CLI: `python -m analyzer` reading stdin; JSON default, readable output option; documented and tested (see ARCHITECTURE.md).
1.9 Benchmark: full pipeline up to 2 MiB; report p50/p95; report-only in CI.
1.10 Docs + DEVELOPMENT_PROGRESS update.
Exit criteria (definitions in PRODUCT_REQUIREMENTS.md): CI green; >=85% line coverage of backend/analyzer/ including the CLI; every redaction pattern tested; no raw log in stdout/stderr/log files; every finding's line numbers match the original log; top-1 accuracy on held-out fixtures reported (>=60% indicative target, 5 cases); benchmark p50/p95 reported (target p95 < 2 s, not a CI gate); CLI behavior documented and tested.

## Phase 2 — API, DB, Auth (needs Supabase free project; data access, storage and retention details deferred)
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
GitHub + Actions (existing, free) · Supabase free project (Phase 2) · hosting deferred (Phase 4) · optional later: Sentry/uptime monitor, domain (paid, needs approval).

## Testing requirements
Unit (rules, redaction, parser) · property/fuzz (redaction never leaks seeded secrets, line count preserved) · golden fixtures (expected category + evidence lines) · API tests (auth, limits, errors, RLS isolation) · frontend component tests + Playwright smoke · CI blocks merge on failure · regression test added for every bug.

## Security requirements
Redact first; raw logs never stored/logged; body logging off; ReDoS-safe bounded regexes + size and per-line caps (no runtime timeout in Phase 1); JWT verification; RLS on all tables; no service-role key in request path; CORS allow-list; security headers; dependency and secret scanning in CI; privacy notice that redaction is best-effort and users should not paste credentials intentionally; user data deletion.

## Git workflow
Feature branches; changes via pull requests; owner reviews and merges. No push, PR creation or merge without explicit owner permission. Docs, scaffold and features go in separate PRs. Meaningful commit messages; history preserved.

## Approved decisions (owner, D1–D9)
- D1 Phase 1 architecture and scope approved.
- D2 Use uv, pin the Python version, commit the lockfile.
- D3 Analyzer first; no database, auth, frontend or external services in Phase 1.
- D4 Synthetic fixtures first; expected results defined before rules; tuning fixtures separate from held-out evaluation; top-1 category explicitly defined.
- D5 No real CI logs committed to the public repo without owner review and sanitization.
- D6 85% line coverage for backend/analyzer/, including the CLI, enforced in CI.
- D7 CLI supports stdin, JSON default, readable output, `__main__.py`; documented and tested.
- D8 Benchmark the full pipeline on up to 2 MiB input; p95 target under 2 s; no nondeterministic timeout in Phase 1.
- D9 PR-based workflow; owner controls merging; no push or merge without explicit permission.
Clarifications: >=20 synthetic fixtures, initial split 15 tuning / 5 held-out (a starting minimum); benchmark is report-only in CI (p50 and p95, no timing-based failures); local `GET /health` stays.

## Deferred (not decided)
Licence · hosting · Phase 2 data-access implementation · Phase 2 retention/storage details · demo audience size.
