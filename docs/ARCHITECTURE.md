# Architecture (FINAL for MVP; changes require owner approval)

## Diagram
Browser (Next.js/TS, Tailwind, shadcn/ui) -> Supabase Auth (JWT) ; Browser -> FastAPI (Bearer JWT)
FastAPI -> analyzer package (pure Python) ; FastAPI -> Supabase Postgres (RLS)

## Repository layout
```
daxtack/
  CLAUDE.md  README.md
  docs/
  backend/
    pyproject.toml
    app/            # FastAPI: main.py, config.py, api/, auth/, repositories/, schemas/
    analyzer/       # pure package: redaction/, parsing/, rules/, ranking/, models.py, cli.py
  frontend/         # Next.js (created in Phase 3)
  tests/
    fixtures/logs/  # sanitized failing logs + expected results (JSON)
    analyzer/  api/  integration/
  .github/workflows/
```
Test location: backend/analyzer/API tests and the shared fixture corpus live in root `tests/` (pytest `testpaths`); frontend tests live inside `frontend/` (Vitest + Testing Library; Playwright E2E later).

## Analyzer pipeline (deterministic, no network, no DB)
1. `redact(text) -> RedactedLog` — per-line, in place, line count preserved; placeholder `[REDACTED:<type>]`; returns counts by type only.
2. `normalize` — strip ANSI/timestamps while keeping an original-line-number map.
3. `segment` — steps/groups, `##[error]`, exit codes.
4. `detect` — rule registry. Rule = id, category, regex (bounded, ReDoS-safe), weight, required/boosting signals, explanation template, fix template.
5. `rank` — score = sum of weights + context boosts (e.g., on failing step, near `##[error]`); confidence derived from score; ties broken deterministically.
6. `AnalysisResult` — findings[{rule_id, category, evidence:[{line, text}], why}], hypotheses[{rank, confidence, explanation, fixes}], redaction_summary.
Future: `Explainer` interface; `NullExplainer` by default; config flag `AI_ENABLED=false`.

## API / persistence decisions
- Backend verifies the Supabase JWT and reads/writes data using the USER's token so Row Level Security is enforced (no service-role key in the request path). Data access sits behind a repository interface (in-memory fake for tests).
- Stored per analysis: redacted log (cap 500 KB), result JSON, hash, counts. Raw input exists only in request memory. Retention default 90 days + user delete (owner to confirm).
- Request-body logging disabled; error handlers never echo input.

## Hosting (undecided, evaluate before Phase 4; verify free-tier and commercial-use terms)
Frontend: Vercel / Cloudflare Pages / Netlify. Backend: Render / Fly.io / Railway. Nothing is deployed without approval.

## Quality tooling
Backend: Python 3.12, ruff, mypy, pytest, pytest-cov, hypothesis (redaction fuzzing), bandit, pip-audit. Frontend: Node LTS, ESLint, Prettier, Vitest, Testing Library, npm audit. Repo: pre-commit + gitleaks, Dependabot. (Exact versions pinned at scaffold time.)
