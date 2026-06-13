# STATE

Single source of "where we are" and "what's next". Read this at session start.

## Current status

- **Session:** 1 (in progress).
- **Build:** module sound (no Go packages yet).
- **Tests:** none yet.
- **Phase 0 (scaffolding):** ✅ committed `e83a81c`.
- **Phase 1 step 1 (schema migration):** ✅ committed `129defd`. Verified against Postgres (11 tables up, 0 down, re-up clean).
- **Phase 1 step 2 (domain state machine):** ✅ committed `890e349`. Exhaustive table-driven tests green (`internal/domain`).
- **Phase 1 step 3 (config loader):** ✅ `internal/config` — env-only, refuses to boot without `DATABASE_URL`/`JWT_SECRET`/`PAYSTACK_SECRET_KEY`, reports all problems at once; tests green.
- **Pushed to remotes:** ✅ Backend repo `origin/main` at `890e349`. Monorepo: PR [#1](https://github.com/Manyle4/mug-e-store/pull/1) (`backend-bootstrap` → `main`, code under `backend/`) — open, awaiting merge.

## Next action

Phase 1 step 4: `internal/auth` — bcrypt (cost 12) hash/compare, HS256 JWT access tokens (15 min, `uid`+`role` claims, signing method asserted on parse), and refresh-token generation + SHA-256 hashing. Tests for token round-trip, expiry, wrong-method rejection, and identical-error semantics. Commit `feat(auth): password hashing and JWT/refresh tokens`.

## Notes / open items

- **Two push targets** (see `DECISIONS.md` 2026-06-13):
  - `origin` = `https://github.com/deladei/coffemug-shop-backend-` — Go code at repo root. Local `main` is based on its existing "Initial commit" (README placeholder preserved).
  - `Manyle4/mug-e-store` — shared monorepo; backend code under `backend/`, delivered by **PR into `main`**, never touching `frontend/`.
- Specs live in `docs/` (moved there from repo root in Session 1).
