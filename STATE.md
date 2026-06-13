# STATE

Single source of "where we are" and "what's next". Read this at session start.

## Current status

- **Session:** 2 (in progress).
- **Build:** `go build ./...` clean.
- **Tests:** `go test ./...` green (domain, config, auth, paystack, sse). `httpapi` + `store` have no tests yet (DB-backed + HTTP-level deferred to Session 4 per plan §5).
- **Phase 0 (scaffolding):** ✅ committed `e83a81c`.
- **Phase 1 step 1 (schema migration):** ✅ committed `129defd`. Verified against Postgres (11 tables up, 0 down, re-up clean).
- **Phase 1 step 2 (domain state machine):** ✅ committed `890e349`. Exhaustive table-driven tests green (`internal/domain`).
- **Phase 1 step 3 (config loader):** ✅ committed `f48982f`. `internal/config` — env-only, refuses to boot without required secrets; tests green.
- **Phase 1 step 4 (auth):** ✅ committed `351c75e`. bcrypt cost 12, HS256 JWT (alg=none rejected), SHA-256 refresh hashing. Deps pinned for Go 1.22.
- **Phase 1 step 5 (store):** ✅ `internal/store` — `store.go` (handle, sentinels, users, refresh tokens), `catalog.go`, `cart.go`, `orders.go` (checkout txn with line snapshotting + cart clear; `TransitionOrder` with `FOR UPDATE` row lock → `domain.CanTransition` → audit event → loyalty earn on completed; ownership returns 404 not 403; idempotency-key NULL-on-empty). Builds + vets clean; **DB-backed tests deferred to Session 4** per plan.
- **Phase 1 step 6 (paystack):** ✅ `internal/paystack` — `Initialize`/`Verify` client (amount in pesewas, currency GHS, baseURL overridable) + `VerifySignature` (constant-time HMAC-SHA512). Tests green: signature valid/forged/wrong-secret/tampered/non-hex, plus Initialize/Verify via httptest.
- **Phase 1 step 7 (sse):** ✅ `internal/sse` — in-process per-order pub/sub broker; non-blocking publish, idempotent leak-free unsubscribe. Passes `-race`.
- **Phase 1 step 8 (httpapi):** ✅ committed `4f75b17`. `internal/httpapi` (10 files) — `server.go` route table + global chain (panic recovery → logging → CORS), `middleware.go` (Bearer/`?token=` auth, role gate, per-IP auth rate limit), handlers for auth/catalog/cart/checkout/orders/SSE/staff/admin, and `webhook_handlers.go` — the Paystack webhook, the **only** path to `paid`, enforcing TRD §5.2's four gates in order (signature → server-side verify → exact amount+GHS → legal transition), idempotent (paid→paid no-op), 5xx→retry / 200→stop, nil system actor. Standard error envelope throughout. The completing piece this session was the webhook handler (prior session stopped mid-step with it the one undefined symbol).
- **Pushed to remotes:** Backend repo `origin/main` at `faa6bc5`; monorepo PR [#1](https://github.com/Manyle4/mug-e-store/pull/1) at `8792eb0`. **paystack + sse + httpapi commits are local, unpushed.**

## Next action

`cmd/api/main.go` — the entrypoint that wires everything into the single runnable binary (plan §2 "get it running"): load `config`, `store.Open(DATABASE_URL)`, build `auth.TokenManager`, `paystack.NewClient`, `sse.Broker`, construct `httpapi.NewServer(...)`, and serve `Handler()` on the configured port with `http.Server` timeouts + graceful shutdown on SIGINT/SIGTERM. No package exists under `cmd/` yet — `NewServer` currently has no caller. After it runs, the next deliverable is the deliberately-deferred **loyalty redemption** feature (plan §3) and then Session 4's DB-backed store + HTTP-level tests (plan §5).

## Notes / open items

- **Two push targets** (see `DECISIONS.md` 2026-06-13):
  - `origin` = `https://github.com/deladei/coffemug-shop-backend-` — Go code at repo root. Local `main` is based on its existing "Initial commit" (README placeholder preserved).
  - `Manyle4/mug-e-store` — shared monorepo; backend code under `backend/`, delivered by **PR into `main`**, never touching `frontend/`.
- Specs live in `docs/` (moved there from repo root in Session 1).
