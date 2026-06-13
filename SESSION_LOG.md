# SESSION LOG

One short entry per session: what was built or changed.

## Session 1 — 2026-06-13

- Bootstrapped the repo. Verified tooling (Go 1.22.4, psql 18.1, git 2.51).
- Moved spec docs into `docs/`; added `.gitignore`, `.env.example`, and the working docs (`STATE.md`, `DECISIONS.md`, `SESSION_LOG.md`).
- Scaffolding commit `e83a81c` (Go module `coffeemug/backend`, git init, remote, doc stubs).
- `feat(db)` `129defd`: schema migration `0001_init`, verified up/down against Postgres.
- `feat(domain)` `890e349`: fulfilment-aware order state machine, exhaustive table-driven tests green.
- Decision: backend pushed to **two** repos — standalone `deladei/coffemug-shop-backend-` (root) and monorepo `Manyle4/mug-e-store` under `backend/` via PR (logged in `DECISIONS.md`).
- Pushed: backend repo `main` ← 3 commits; monorepo PR #1 opened (`backend-bootstrap` → `main`).
