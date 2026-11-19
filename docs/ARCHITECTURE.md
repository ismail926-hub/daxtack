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
    analyzer/       # pure package: redaction/, parsing/, rules/, ranking/, models.py, cli.py, __main__.py
  frontend/         # Next.js (created in Phase 3)
  tests/
    fixtures/logs/  # synthetic logs + expected.json: tuning/ and heldout/ (no real logs without owner review)
    analyzer/  api/  integration/   # api/ and integration/ arrive in Phase 2
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

## API / persistence decisions — PROVISIONAL (Phase 2; data access, storage and retention are deferred, not approved)
- Backend verifies the Supabase JWT and reads/writes data using the USER's token so Row Level Security is enforced (no service-role key in the request path). Data access sits behind a repository interface (in-memory fake for tests).
- Stored per analysis: redacted log (cap 500 KB), result JSON, hash, counts. Raw input exists only in request memory. Retention default 90 days + user delete (owner to confirm).
- Request-body logging disabled; error handlers never echo input.

## Hosting (deferred; undecided, evaluate before Phase 4; verify free-tier and commercial-use terms)
Frontend: Vercel / Cloudflare Pages / Netlify. Backend: Render / Fly.io / Railway. Nothing is deployed without approval.

## CLI (Phase 1)
`python -m analyzer` (via `analyzer/__main__.py`) reads the log from stdin. JSON output is the default; a readable text format is selectable (flag name fixed in the CLI PR). Exit codes and the handling of empty, oversize (> MAX_LOG_BYTES) and invalid-UTF-8 input are specified and tested before implementation. The CLI never echoes raw input; errors go to stderr without log content.

## Phase 1 safety bounds
Deterministic limits only: input size cap, per-line length cap and bounded regexes. No runtime timeout in Phase 1; any time budget is reconsidered with the API in Phase 2.

## Quality tooling
Backend: uv with a pinned Python version (`.python-version` and `requires-python`; the 3.12 line is planned, exact version confirmed at scaffold time) and a committed `uv.lock` (CI installs with the lockfile enforced); ruff, mypy, pytest, pytest-cov (85% gate for `backend/analyzer/`), hypothesis (redaction fuzzing), bandit, pip-audit. Frontend (Phase 3): Node LTS, ESLint, Prettier, Vitest, Testing Library, npm audit. Repo: pre-commit + gitleaks (also run in CI), Dependabot. Benchmark runs in CI as report-only.
