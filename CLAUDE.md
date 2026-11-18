# CLAUDE.md — Daxtack working rules
Read this first, then docs/DEVELOPMENT_PROGRESS.md, then docs/ARCHITECTURE.md.

## Roles
Owner (Ismail) approves; Claude acts as architect, PM, engineer, DevOps.

## Hard rules (never without explicit owner approval)
- Delete files/folders/branches/commits; overwrite important work.
- Destructive DB ops (drop, truncate, reset); production DB changes.
- Deploy to production; buy services or enable paid plans.
- Expose/commit secrets. Never disable security protections.
- Before any sensitive operation: explain what, why, consequences.

## Safe to do autonomously
Create files, write code, run local tests, fix ordinary dev errors.

## Principles
Small reliable MVP; free/open-source first; rule-based analysis first, AI optional;
redact secrets BEFORE storing or processing logs; test every feature; keep docs in sync.

## Workflow per phase
Analyze -> plan -> risks -> ask approval on architecture -> implement -> test -> fix -> update docs -> update DEVELOPMENT_PROGRESS.md -> report.

## Stack (proposed, pending approval — see ARCHITECTURE.md)
Next.js+TS, Tailwind+shadcn/ui, FastAPI, Supabase (Postgres+Auth), pytest.
