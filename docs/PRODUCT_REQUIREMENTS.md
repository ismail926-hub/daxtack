# Product Requirements

## Competitors (searched Nov 2026; verify pricing before positioning)
SenseTheLog (Action + AI, ~100 free analyses/mo) · FailBrief (GitHub App, AI PR summaries, free plan) · CodeRabbit (PR review incl. CI log analysis) · Agnox (AI triage for test automation) · indie tools (PipelineIQ, Claude-to-Slack summarizer, Ollama scripts).
Gaps: no-install paste workflow, deterministic/explainable analysis, redaction-first privacy, self-hostability. Risk: low barrier to entry.

## MVP functional requirements
FR1 Paste a GitHub Actions log (max 2 MiB = 2,097,152 bytes, UTF-8).
FR2 Redact secrets before any processing, logging or storage; report redaction count (never the values).
FR3 Parse: strip ANSI/timestamps, detect groups/steps, `##[error]` annotations, exit codes. Line numbers always refer to the ORIGINAL pasted log.
FR4 Detect error types and failure patterns; each finding cites rule id, category, evidence lines with line numbers.
FR5 Root cause: ranked hypotheses with confidence and "why" (matched rules + evidence).
FR6 Suggested fixes: steps plus YAML/code snippets tied to the matched rule.
FR7 History: save/list/view/delete own analyses (redacted data only).
FR8 Dashboard: usage vs limit, account info.
FR9 Auth: register/login/logout via Supabase Auth.

## Non-functional requirements
NFR1 Raw logs never persisted or logged. NFR2 Full-pipeline analysis p95 < 2 s for inputs up to 2 MiB (target; see Definitions). NFR3 Deterministic output (same input -> same result). NFR4 Analyzer has no network/DB dependency. NFR5 AI disabled by default. NFR6 >= 85% line coverage for `backend/analyzer/`, including the CLI, enforced in CI.

## Initial rule categories
Dependency install (npm/pip/yarn/maven) · missing module/import · test failures (jest/pytest/junit) · build/compile · Docker build · auth/permissions (401/403, token scope, missing secrets) · timeout/OOM/disk · network/runner flakiness · workflow YAML syntax · version mismatch (Node/Python) · cache problems.

## Out of scope for MVP
GitHub App/Action, Slack, LLMs, teams/orgs, billing, other CI providers.

## Success metrics
- Phase 1: top-1 accuracy on the held-out synthetic fixtures (see Definitions); with only 5 held-out cases this is an indicative check, not statistical validation.
- Post-Phase 1 product targets: >=70% correct top-1 category on a larger corpus of real, owner-reviewed and sanitized logs; 10 returning users (Phase 5).

## Definitions (Phase 1)
- **Fixture:** a synthetic log plus an `expected.json` (expected category and evidence line numbers). Expected results are written and committed BEFORE the rules they test. Minimum 20 fixtures, initially 15 tuning (`tests/fixtures/logs/tuning/`) and 5 held-out (`tests/fixtures/logs/heldout/`). Held-out fixtures are never used for tuning; if one drives a rule change it moves to tuning and is replaced.
- **Real logs:** none are committed to this public repository without owner review and sanitization.
- **Top-1 category:** the highest-ranked hypothesis's category equals the fixture's expected category (ties broken deterministically). If nothing is detected, that counts as correct only when the expected category is `unknown`. Reported on held-out fixtures.
- **Coverage:** line coverage of `backend/analyzer/` including the CLI, enforced in CI at 85%.
- **Benchmark:** full pipeline (redact through result) on synthetic inputs up to 2 MiB; report p50 and p95 with hardware noted. Target p95 < 2 s. Report-only in CI: timing never fails CI. No runtime timeout in Phase 1.
