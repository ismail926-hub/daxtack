# Project Overview
**Problem:** CI logs are long and noisy; developers lose time finding the real error and the fix.
**Product:** Daxtack analyzes GitHub Actions failure logs and returns error type, evidence lines (with line numbers), probable root cause, and suggested fixes.
**Differentiation:** deterministic, explainable rule engine (no LLM cost); secret redaction first; paste-and-go (no GitHub App permissions); generous free tier; optional AI later.
**Market reality:** AI-first competitors exist (SenseTheLog, FailBrief, CodeRabbit, Agnox, indie tools). Daxtack competes on privacy, explainability and rule quality, not on "uses AI".
**Target users:** students, solo developers, small teams, junior DevOps.
**Business model (later):** free monthly quota; paid for GitHub integration, team history, optional AI explanations.
**Validation:** before heavy investment, show the working analyzer (CLI/demo) to ~10 developers (see IMPLEMENTATION_PLAN.md, Phase 1 exit).
