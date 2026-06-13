# DECISIONS

Every deviation from a spec or a pinned choice, with the reason. Append-only.

| Date | Decision | Reason |
|------|----------|--------|
| 2026-06-13 | Moved the spec docs from the repo root into `docs/`. | `CLAUDE.md` §2 and the kickoff reference specs as `docs/coffee-mug-shop-*.md`; this makes those paths resolve instead of diverging. No spec content changed. |
| 2026-06-13 | Pinned `golang.org/x/crypto` to **v0.31.0** and kept the `go` directive at **1.22** (build with `GOTOOLCHAIN=local`). | `x/crypto` ≥ v0.53.0 requires Go 1.25; `go mod tidy` silently bumped the module to `go 1.25.0` and pulled a 1.25 toolchain on the fly. That contradicts the pinned "Go 1.22" stack (CLAUDE.md §5) and the installed 1.22.4. v0.31.0 is the newest release that builds on 1.22. `github.com/golang-jwt/jwt/v5` and `golang.org/x/crypto/bcrypt` themselves are part of the pinned stack, so adding them needs no separate approval. |
| 2026-06-13 | Backend code is pushed to **two** repos, overriding Git Law §3.4 ("backend code goes to this repo only"). Targets: (a) standalone `deladei/coffemug-shop-backend-` with the Go code at repo root; (b) shared monorepo `deladei`→`Manyle4/mug-e-store` under a `backend/` subfolder, alongside the existing `frontend/`, delivered by **PR into `main`** (never a force-push, never touching `frontend/`). | Explicit owner instruction (2026-06-13): the team treats `Manyle4/mug-e-store` as a shared monorepo, not the frontend team's private repo, and the `backend/` folder is expected to live there next to `frontend/`. `CLAUDE.md` §3.4–3.5 will be reconciled to this in a follow-up edit. |
