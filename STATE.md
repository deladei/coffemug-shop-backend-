# STATE

Single source of "where we are" and "what's next". Read this at session start.

## Current status

- **Session:** 1 (in progress).
- **Build:** module sound (no Go packages yet).
- **Tests:** none yet.
- **Phase 0 (scaffolding):** ✅ committed `e83a81c`.
- **Phase 1 step 1 (schema migration):** ✅ committed `129defd`. `migrations/0001_init.{up,down}.sql` verified against Postgres (11 tables up, 0 down, re-up clean).

## Next action

Phase 1 step 2: `internal/domain` — the order status type, the fulfilment-aware transition function, terminality, and status validation. **Write the table-driven tests first**, covering every legal and illegal transition for both pickup and delivery. Then the code. `go build`/`vet`/`test` must be green. Commit `feat(domain): order state machine with tests`.

## Notes / open items

- **Two push targets** (see `DECISIONS.md` 2026-06-13):
  - `origin` = `https://github.com/deladei/coffemug-shop-backend-` — Go code at repo root. Local `main` is based on its existing "Initial commit" (README placeholder preserved).
  - `Manyle4/mug-e-store` — shared monorepo; backend code under `backend/`, delivered by **PR into `main`**, never touching `frontend/`.
- Specs live in `docs/` (moved there from repo root in Session 1).
