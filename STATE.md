# STATE

Single source of "where we are" and "what's next". Read this at session start.

## Current status

- **Session:** 1 (in progress) — repo bootstrap.
- **Build:** not yet (no Go code).
- **Tests:** none yet.
- **Phase 0 (scaffolding):** in progress — tooling verified (Go 1.22.4, psql 18.1, git 2.51), docs moved into `docs/`, `.gitignore` + `.env.example` + working docs created.

## Next action

Complete Phase 0: `git init` (branch `main`), add `origin` remote, run `git ls-remote` to confirm access and that the remote is empty, `go mod init coffeemug/backend`, then make the scaffolding commit. After that, begin Phase 1 step 1: `migrations/0001_init.up.sql` + `.down.sql` built exactly from `docs/coffee-mug-shop-database-schema.md`.

## Notes / open items

- **Two push targets** (see `DECISIONS.md` 2026-06-13):
  - `origin` = `https://github.com/deladei/coffemug-shop-backend-` — Go code at repo root. Local `main` is based on its existing "Initial commit" (README placeholder preserved).
  - `Manyle4/mug-e-store` — shared monorepo; backend code under `backend/`, delivered by **PR into `main`**, never touching `frontend/`.
- Specs live in `docs/` (moved there from repo root in Session 1).
