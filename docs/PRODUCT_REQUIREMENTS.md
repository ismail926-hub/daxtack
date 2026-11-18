# Product Requirements

## Competitors (searched Nov 2026; verify pricing before positioning)
SenseTheLog (Action + AI, ~100 free analyses/mo) · FailBrief (GitHub App, AI PR summaries, free plan) · CodeRabbit (PR review incl. CI log analysis) · Agnox (AI triage for test automation) · indie tools (PipelineIQ, Claude-to-Slack summarizer, Ollama scripts).
Gaps: no-install paste workflow, deterministic/explainable analysis, redaction-first privacy, self-hostability. Risk: low barrier to entry.

## MVP functional requirements
FR1 Paste a GitHub Actions log (max 2 MB, UTF-8).
FR2 Redact secrets before any processing, logging or storage; report redaction count (never the values).
FR3 Parse: strip ANSI/timestamps, detect groups/steps, `##[error]` annotations, exit codes. Line numbers always refer to the ORIGINAL pasted log.
FR4 Detect error types and failure patterns; each finding cites rule id, category, evidence lines with line numbers.
FR5 Root cause: ranked hypotheses with confidence and "why" (matched rules + evidence).
FR6 Suggested fixes: steps plus YAML/code snippets tied to the matched rule.
FR7 History: save/list/view/delete own analyses (redacted data only).
FR8 Dashboard: usage vs limit, account info.
FR9 Auth: register/login/logout via Supabase Auth.

## Non-functional requirements
NFR1 Raw logs never persisted or logged. NFR2 Analysis p95 < 2 s for 2 MB. NFR3 Deterministic output (same input -> same result). NFR4 Analyzer has no network/DB dependency. NFR5 AI disabled by default. NFR6 Analyzer test coverage >= 85%.

## Initial rule categories
Dependency install (npm/pip/yarn/maven) · missing module/import · test failures (jest/pytest/junit) · build/compile · Docker build · auth/permissions (401/403, token scope, missing secrets) · timeout/OOM/disk · network/runner flakiness · workflow YAML syntax · version mismatch (Node/Python) · cache problems.

## Out of scope for MVP
GitHub App/Action, Slack, LLMs, teams/orgs, billing, other CI providers.

## Success metrics
>=70% correct top-1 category on a corpus of 50+ real logs; 10 returning users.
