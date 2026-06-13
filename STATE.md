# STATE

Single source of "where we are" and "what's next". Read this at session start.

## Current status

- **Session:** 1 (in progress).
- **Build:** module sound (no Go packages yet).
- **Tests:** none yet.
- **Phase 0 (scaffolding):** ✅ committed `e83a81c`.
- **Phase 1 step 1 (schema migration):** ✅ committed `129defd`. Verified against Postgres (11 tables up, 0 down, re-up clean).
- **Phase 1 step 2 (domain state machine):** ✅ committed `890e349`. Exhaustive table-driven tests green (`internal/domain`).
- **Phase 1 step 3 (config loader):** ✅ committed `f48982f`. `internal/config` — env-only, refuses to boot without required secrets; tests green.
- **Phase 1 step 4 (auth):** ✅ `internal/auth` — bcrypt cost 12, HS256 JWT (15 min, `uid`+`role`, signing method asserted, alg=none rejected), SHA-256 refresh-token hashing. Tests green. Deps pinned for Go 1.22 (`x/crypto v0.31.0`, `jwt v5.3.1`).
- **Pushed to remotes:** Backend repo `origin/main` at `890e349` (commits `f48982f` + auth are local, unpushed). Monorepo: PR [#1](https://github.com/Manyle4/mug-e-store/pull/1) open.

## Next action

Phase 1 step 5: `internal/store` — `store.go` (db handle, error sentinels, users, refresh tokens), `catalog.go`, `cart.go`, `orders.go` (checkout as a single transaction with line snapshotting + cart clear; transitions with a row lock, audit event, and loyalty earn on completion). Build against the schema. Commit `feat(store): persistence layer`. Note: store tests against a real DB are Session 4 per the plan — keep this layer compiling + vet-clean now.

## Notes / open items

- **Two push targets** (see `DECISIONS.md` 2026-06-13):
  - `origin` = `https://github.com/deladei/coffemug-shop-backend-` — Go code at repo root. Local `main` is based on its existing "Initial commit" (README placeholder preserved).
  - `Manyle4/mug-e-store` — shared monorepo; backend code under `backend/`, delivered by **PR into `main`**, never touching `frontend/`.
- Specs live in `docs/` (moved there from repo root in Session 1).
