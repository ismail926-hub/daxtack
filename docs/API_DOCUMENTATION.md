# API — base `/api/v1`, auth `Authorization: Bearer <Supabase JWT>` (Phase 2)
Phase 1 exposes only `GET /health` (and the analyzer via CLI).
- POST /analyses {log, title?} -> 201 AnalysisResult (redacted, analyzed, saved, usage incremented)
- GET /analyses?limit&cursor · GET /analyses/{id} · DELETE /analyses/{id} (204)
- GET /me/usage -> {used, limit, period}
- POST /analyze/preview {log} -> optional, unauthenticated, rate-limited, not saved
Errors: {error:{code,message}} — 413 oversize, 422 invalid, 429 limit. Messages never echo log content.
