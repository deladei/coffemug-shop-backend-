# STATE

Single source of "where we are" and "what's next". Read this at session start.

## Current status

- **Session:** 2 (in progress).
- **Runnable:** ✅ `cmd/api` binary builds and boots; fails fast with an aggregated config error when required env is unset.
- **Loyalty redemption:** ✅ feature complete (commit pending push). Pure `domain.Redemption` rule (unit-tested, 9 cases); `store.Checkout` redeems inside the txn under a user-row `FOR UPDATE` lock; `TransitionOrder` refunds on cancel. Contract change published to the API brief. **DB-backed redemption tests (happy-path, over-balance 409, concurrent double-spend) deferred to Session 4** per the established no-test-DB-this-session pattern.
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
- **Phase 1 step 9 (cmd/api):** ✅ committed `9b8e778`. `cmd/api/main.go` — composition + lifecycle only: `config.LoadFromEnv` → `store.Open` + startup ping (fail fast) → `auth.NewTokenManager`, `paystack.NewClient`, `sse.NewBroker` → `httpapi.NewServer` → serve `Handler()`. `http.Server` with slowloris read timeouts but **`WriteTimeout: 0`** (a positive write deadline would sever live SSE streams); SIGINT/SIGTERM → bounded graceful `Shutdown`. Binary boots + fails fast on missing env.
- **Pushed to remotes:** Backend repo `origin/main` at `9b8e778` (paystack + sse were already on origin; this session pushed httpapi + cmd/api). Monorepo PR [#1](https://github.com/Manyle4/mug-e-store/pull/1) at `8792eb0` — **NOT yet updated with httpapi/cmd; needs a PR-mirror pass** (backend code under `backend/`, never touching `frontend/`).

## Next action

**Session 4 — the tests this sandbox has deferred (plan §5), against a real test database.** Stand up a Postgres test DB + a store test harness, then cover, in priority order: (1) `store.Checkout` — happy path, empty cart, unavailable item, duplicate idempotency key, **and the three redemption cases plan §3 calls for: redeem exactly the balance; over-balance → `ErrInsufficientPoints`; two concurrent redemptions of the same points → only one commits** (this is the double-spend test the `FOR UPDATE` user-row lock exists to pass); (2) `TransitionOrder` — legal/illegal/idempotent no-op, loyalty earn on completion, **and `refund_on_cancel` when a redeemed order is cancelled**; (3) HTTP-level tests with `httptest` — auth flow, ownership `404`, staff `403` on manual `paid`, and the webhook four-gate flow with a faked signature. After that: user-owned real-Paystack E2E (§4) + deploy (§6).

**Monorepo PR — up to date.** `Manyle4/mug-e-store` PR #1 (`backend-bootstrap` → `main`) mirrored to standalone `3a9d3db` at monorepo commit `5df0579` (`backend/` only; `frontend/` untouched, verified 0 frontend files in the PR diff). Includes httpapi, cmd/api, loyalty redemption, and the API-brief contract change. Re-run a mirror pass after the next backend milestone.

## Notes / open items

- **Two push targets** (see `DECISIONS.md` 2026-06-13):
  - `origin` = `https://github.com/deladei/coffemug-shop-backend-` — Go code at repo root. Local `main` is based on its existing "Initial commit" (README placeholder preserved).
  - `Manyle4/mug-e-store` — shared monorepo; backend code under `backend/`, delivered by **PR into `main`**, never touching `frontend/`.
- Specs live in `docs/` (moved there from repo root in Session 1).
