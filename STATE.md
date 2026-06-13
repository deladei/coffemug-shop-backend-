# STATE

Single source of "where we are" and "what's next". Read this at session start.

## Current status

- **Session:** 2 (in progress).
- **Runnable:** ✅ `cmd/api` binary builds and boots; fails fast with an aggregated config error when required env is unset.
- **Loyalty redemption:** ✅ feature complete + DB-tested. Pure `domain.Redemption` rule (9 unit cases); `store.Checkout` redeems inside the txn under a user-row `FOR UPDATE` lock; `TransitionOrder` refunds on cancel. Contract published to the API brief.
- **Build:** `go build ./...` clean; `go vet ./...` clean.
- **Tests:** `go test ./...` green; **with `TEST_DATABASE_URL` set, `go test -p 1 ./...` green** (28 tests incl. DB-backed store + HTTP-level). Without the var the store/httpapi DB suites skip, so default `go test ./...` stays green on a box without Postgres.
- **Test DB:** local Postgres 18 on socket; created `coffeemug_test` (owner `walker`) with `0001_init` applied. DSN used: `host=/var/run/postgresql user=walker dbname=coffeemug_test sslmode=disable` (peer auth — TCP needs a password, the socket does not).
- **Phase 0 (scaffolding):** ✅ committed `e83a81c`.
- **Phase 1 step 1 (schema migration):** ✅ committed `129defd`. Verified against Postgres (11 tables up, 0 down, re-up clean).
- **Phase 1 step 2 (domain state machine):** ✅ committed `890e349`. Exhaustive table-driven tests green (`internal/domain`).
- **Phase 1 step 3 (config loader):** ✅ committed `f48982f`. `internal/config` — env-only, refuses to boot without required secrets; tests green.
- **Phase 1 step 4 (auth):** ✅ committed `351c75e`. bcrypt cost 12, HS256 JWT (alg=none rejected), SHA-256 refresh hashing. Deps pinned for Go 1.22.
- **Phase 1 step 5 (store):** ✅ `internal/store` — `store.go` (handle, sentinels, users, refresh tokens), `catalog.go`, `cart.go`, `orders.go` (checkout txn with line snapshotting + cart clear; `TransitionOrder` with `FOR UPDATE` row lock → `domain.CanTransition` → audit event → loyalty earn on completed; ownership returns 404 not 403; idempotency-key NULL-on-empty). Builds + vets clean; **DB-backed tests deferred to Session 4** per plan.
- **Phase 1 step 6 (paystack):** ✅ `internal/paystack` — `Initialize`/`Verify` client (amount in pesewas, currency GHS, baseURL overridable) + `VerifySignature` (constant-time HMAC-SHA512). Tests green: signature valid/forged/wrong-secret/tampered/non-hex, plus Initialize/Verify via httptest.
- **Phase 1 step 7 (sse):** ✅ `internal/sse` — in-process per-order pub/sub broker; non-blocking publish, idempotent leak-free unsubscribe. Passes `-race`.
- **Phase 1 step 8 (httpapi):** ✅ committed `4f75b17`. `internal/httpapi` (10 files) — `server.go` route table + global chain (panic recovery → logging → CORS), `middleware.go` (Bearer/`?token=` auth, role gate, per-IP auth rate limit), handlers for auth/catalog/cart/checkout/orders/SSE/staff/admin, and `webhook_handlers.go` — the Paystack webhook, the **only** path to `paid`, enforcing TRD §5.2's four gates in order (signature → server-side verify → exact amount+GHS → legal transition), idempotent (paid→paid no-op), 5xx→retry / 200→stop, nil system actor. Standard error envelope throughout. The completing piece this session was the webhook handler (prior session stopped mid-step with it the one undefined symbol).
- **Phase 1 step 9 (cmd/api):** ✅ committed `9b8e778`. `cmd/api/main.go` — composition + lifecycle only: `config.LoadFromEnv` → `store.Open` + startup ping (fail fast) → `auth.NewTokenManager`, `paystack.NewClient`, `sse.NewBroker` → `httpapi.NewServer` → serve `Handler()`. `http.Server` with slowloris read timeouts but **`WriteTimeout: 0`** (a positive write deadline would sever live SSE streams); SIGINT/SIGTERM → bounded graceful `Shutdown`. Binary boots + fails fast on missing env.
- **Phase 1 step 10 (tests):** ✅ committed `8124f79` (store) + `013e4bb` (httpapi). Store suite against real Postgres — checkout (happy/empty/unavailable/duplicate-key), redemption (exact balance / over-balance reject / **concurrent double-spend → exactly one commit, balance 0, passes `-race`**), transition (legal/illegal/no-op/earn/refund-on-cancel). HTTP suite via `httptest` + fake Paystack — auth flow, wrong-password 401, ownership 404, staff 403 on manual `paid`, and the webhook four-gate matrix (happy/bad-sig/verify-failed/amount-mismatch/idempotent). Both skip without `TEST_DATABASE_URL`.
- **Pushed to remotes:** Backend repo `origin/main` at `9b8e778` (then `+` loyalty + tests this session, **push pending** — see below). Monorepo PR #1 mirrored to `3a9d3db` (`5df0579`) — **now behind again** by loyalty + tests; needs a re-mirror.

## Next action

**Backend code + tests for Phase 1 are complete.** What remains is owner-only (cannot be done from this sandbox):
1. **Real-Paystack E2E (plan §4):** put a Paystack **test** secret key in `.env`, tunnel the local server (e.g. `ngrok http 8080`), register the webhook URL, run a real checkout with a test card, and confirm the order flips to `paid` over the live webhook + the SSE stream updates. Also verify a *pending* (not-success) payment does **not** flip the order.
2. **Deploy (plan §6):** host backend + Postgres; set `FRONTEND_ORIGIN`; switch the refresh cookie to `SameSite=None; Secure` if cross-origin.

If continuing in-sandbox instead: re-mirror the monorepo PR (now behind by loyalty + tests), or start Phase 2 features.

**Monorepo PR — behind again.** `Manyle4/mug-e-store` PR #1 was last mirrored to `3a9d3db` (`5df0579`); it now lacks the loyalty + test commits (`74376c9`, `8124f79`, `013e4bb`, docs). Re-run the mirror pass (clone, `git archive HEAD | tar -x -C backend/`, verify 0 `frontend/` files, push `backend-bootstrap`).

## Notes / open items

- **Two push targets** (see `DECISIONS.md` 2026-06-13):
  - `origin` = `https://github.com/deladei/coffemug-shop-backend-` — Go code at repo root. Local `main` is based on its existing "Initial commit" (README placeholder preserved).
  - `Manyle4/mug-e-store` — shared monorepo; backend code under `backend/`, delivered by **PR into `main`**, never touching `frontend/`.
- Specs live in `docs/` (moved there from repo root in Session 1).
