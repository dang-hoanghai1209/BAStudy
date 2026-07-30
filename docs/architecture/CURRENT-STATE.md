# Current State of the Repository

**Status:** Verified — every statement below was confirmed by direct
inspection of this repository during the VS-000 audit (2026-07-26).
**Scope:** what exists *today*. For where the project is going, see
[`TARGET-ARCHITECTURE.md`](TARGET-ARCHITECTURE.md). Nothing may be moved
from that document into this one without repository evidence.

## 1. What this repository is

SellerMargin is a **fork of the Full Stack FastAPI Template repository
itself**, not a project generated from it with Copier.

Evidence:

- `git rev-list --count HEAD` → 1392 commits, all upstream template history
- [`copier.yml`](../../copier.yml) (the template *definition*) is present at
  the repository root
- [`.copier/.copier-answers.yml.jinja`](../../.copier/.copier-answers.yml.jinja)
  exists, but a rendered `.copier-answers.yml` does **not**
- [`package.json`](../../package.json) is still named `fastapi-full-stack-template`
- No file in the tracked tree mentions "SellerMargin"

Consequence: `copier update` is not an available upgrade path. Upstream
changes must be tracked as git history. See
[`adr/ADR-0005-UPSTREAM-TRACKING.md`](adr/ADR-0005-UPSTREAM-TRACKING.md).

At the time of the audit `origin` pointed at
`https://github.com/fastapi/full-stack-fastapi-template.git`; it has since
been corrected to `https://github.com/dang-hoanghai1209/BAStudy.git`.

## 2. Branches

- Default branch: **`master`** (not `main`)
- Documentation work happens on `docs/VS-000-master-plan`
- Other remote branches are upstream template branches (dependabot, etc.)

## 3. Backend

**Stack:** FastAPI, SQLModel, Alembic, PostgreSQL, Pydantic v2 / pydantic-settings,
PyJWT, pwdlib (Argon2 + bcrypt), Sentry SDK. Python `>=3.14` per
[`backend/pyproject.toml`](../../backend/pyproject.toml); `.python-version` pins `3.14`.

**Package manager:** `uv`, as a workspace. `[tool.uv.workspace] members = ["backend"]`
in the root [`pyproject.toml`](../../pyproject.toml). The virtualenv lives at the
**repository root** (`.venv/`), not at `backend/.venv/`.

**Structure is flat, not modular.** There is no `backend/app/modules/`
directory. The actual layout:

```
backend/app/
├── main.py               FastAPI app, CORS, Sentry, router mount at /api/v1
├── models.py             ALL SQLModel models (User, Item) in one file
├── crud.py               ALL data access helpers in one file
├── utils.py              email helpers, password-reset tokens
├── initial_data.py       first-superuser seeding
├── backend_pre_start.py  DB readiness wait
├── tests_pre_start.py    DB readiness wait for tests
├── api/
│   ├── main.py           APIRouter aggregation
│   ├── deps.py           SessionDep, CurrentUser, get_current_active_superuser
│   └── routes/           items.py, login.py, private.py, users.py, utils.py
├── core/
│   ├── config.py         Settings (pydantic-settings, reads ../.env)
│   ├── db.py             engine + init_db
│   └── security.py       JWT create, password hash/verify
└── alembic/              env.py, script.py.mako, versions/
```

Moving to a module layout is a decision, not a fact — see
[`adr/ADR-0009-BACKEND-MODULE-STRUCTURE.md`](adr/ADR-0009-BACKEND-MODULE-STRUCTURE.md).

### Backend commands (verified)

| Purpose | Command | Notes |
|---|---|---|
| Install | `uv sync --all-packages` | from repo root |
| Run (local) | `fastapi run --reload backend/app/main.py` | entrypoint declared as `app.main:app` in `backend/pyproject.toml` |
| Lint + typecheck | `cd backend && uv run bash scripts/lint.sh` | runs `mypy app`, `ty check app`, `ruff check app`, `ruff format --check` |
| Format | `cd backend && uv run bash scripts/format.sh` | `ruff check --fix` + `ruff format` |
| Migrate + seed | `cd backend && uv run bash scripts/prestart.sh` | waits for DB, `alembic upgrade head`, `python app/initial_data.py` |
| Test | `cd backend && uv run bash scripts/tests-start.sh` | pre-start wait, then `scripts/test.sh` |
| Test (DB already up) | `cd backend && uv run bash scripts/test.sh` | `coverage run -m pytest tests/` + report + html |

`mypy` is configured `strict = true`; `ruff` targets `py314` and forbids
`print` (`T201`). Both exclude `alembic`.

## 4. Data model and migrations

**Only two tables exist**, both from the template
([`backend/app/models.py`](../../backend/app/models.py)):

| Table | Columns of note |
|---|---|
| `user` | `id` UUID PK, `email` unique indexed, `hashed_password`, `is_active`, `is_superuser`, `full_name`, `created_at` (timezone-aware) |
| `item` | `id` UUID PK, `title`, `description`, `created_at`, `owner_id` FK → `user.id` `ON DELETE CASCADE` |

There is **no** organization, shop, product, SKU, order, cost, fee,
import, settlement or audit entity of any kind. There are no money columns
anywhere in the schema.

**Alembic** is configured with `script_location = app/alembic` in
[`backend/alembic.ini`](../../backend/alembic.ini); the URL is injected at runtime by
`env.py` from settings (not stored in the ini). The revision chain is
linear with a single head:

```
e2412789c190  initialize models          (down_revision = None)
  → 9c0a54914c78  add max length for string/varchar
    → d98dd8ec85a3  replace id integers in all models
      → 1a31ce608336  add cascade delete relationships
        → fe56fa70289e  add created_at to user and item   ← HEAD
```

## 5. API surface (current)

All routes are mounted under `settings.API_V1_STR` = **`/api/v1`**.
`generate_unique_id_function` produces `{tag}-{name}` operation IDs, which
is what gives the generated TypeScript client its `XService.methodName` shape.

| Router | Endpoints |
|---|---|
| `login` | `POST /login/access-token`, `POST /login/test-token`, `POST /password-recovery/{email}`, `POST /reset-password/`, `POST /password-recovery-html-content/{email}` (superuser) |
| `users` | `GET /users/` (superuser), `POST /users/` (superuser), `GET|PATCH|DELETE /users/me`, `PATCH /users/me/password`, `POST /users/signup`, `GET /users/{user_id}`, `PATCH /users/{user_id}` (superuser), `DELETE /users/{user_id}` (superuser) |
| `utils` | `POST /utils/test-email/` (superuser), `GET /utils/health-check/` |
| `items` | `GET /items/`, `GET /items/{id}`, `POST /items/`, `PUT /items/{id}`, `DELETE /items/{id}` |
| `private` | `POST /private/users/` — **only mounted when `ENVIRONMENT == "local"`** |

Schemas are SQLModel classes co-located in `models.py`: `*Base`, `*Create`,
`*Update`, `*Public`, `*sPublic` (list + count), plus `Message`, `Token`,
`TokenPayload`, `NewPassword`.

## 6. Authentication and authorization (current)

**Authentication:** OAuth2 password flow with a bearer JWT.
`create_access_token` signs `{sub: <user id>, exp}` with **HS256** using
`settings.SECRET_KEY`; default expiry is 8 days
(`ACCESS_TOKEN_EXPIRE_MINUTES = 60 * 24 * 8`). Passwords are hashed with
pwdlib using Argon2 with a bcrypt fallback, and `verify_and_update`
transparently rehashes on login. `crud.authenticate` verifies against a
dummy hash when the user is not found, to blunt timing attacks.
Password recovery returns an identical response whether or not the email
exists, to avoid enumeration.

**Authorization is two-level and has no tenancy:**

1. `get_current_user` — valid token, user exists, user is active
2. `get_current_active_superuser` — additionally requires `is_superuser`

Resource-level access is ad hoc: `items` endpoints compare
`item.owner_id != current_user.id` inline in each handler. There is **no
organization, no membership, no role table, and no shared scoping
dependency**. Every business table SellerMargin adds will need one — see
[`../security/AUTHORIZATION-POLICY.md`](../security/AUTHORIZATION-POLICY.md).

The frontend stores the access token in `localStorage`
(`frontend/src/hooks/useAuth.ts`) and guards the admin route by checking
`is_superuser` in a `beforeLoad`; that is a UI affordance only, and the
server check is the real gate.

## 7. Frontend

**Stack:** React 19, Vite 8, TypeScript 5.9, TanStack Router (file-based
routes, `autoCodeSplitting`), TanStack Query 5, TanStack Table 8,
Tailwind CSS 4, Radix UI primitives with shadcn-style components in
`src/components/ui/`, `react-hook-form` + `zod`, `axios`, `sonner` toasts,
`next-themes`. Linting and formatting is **Biome**, not ESLint/Prettier.

**Package manager: Bun** (workspace declared in the root `package.json`,
`bun.lock` at root).

```
frontend/src/
├── client/          GENERATED from OpenAPI — never hand-edit
├── components/      Admin/, Common/, Items/, Pending/, Sidebar/, UserSettings/, ui/
├── hooks/           useAuth, useCustomToast, useCopyToClipboard, useMobile
├── routes/          __root.tsx, _layout.tsx, _layout/{index,items,admin,settings}.tsx,
│                    login.tsx, signup.tsx, recover-password.tsx, reset-password.tsx
├── routeTree.gen.ts GENERATED by the TanStack Router Vite plugin — never hand-edit
├── main.tsx, utils.ts, index.css
└── tests/           Playwright specs (not unit tests)
```

Existing screens are the template's: login, signup, password recovery/reset,
a placeholder dashboard (`_layout/index.tsx` renders only a greeting), items
CRUD, user settings, and a superuser-only admin user list. Page titles still
read "FastAPI Template".

### Frontend commands (verified)

| Purpose | Command |
|---|---|
| Install | `bun ci` (root or `frontend/`) |
| Dev server | `bun run dev` → Vite on `http://localhost:5173` |
| Build | `bun run --filter frontend build` (`tsc -p tsconfig.build.json && vite build`) |
| Lint/format | `bun run lint` → `biome check --write --unsafe ...` (**writes files**) |
| Regenerate API client | `bun run --filter frontend generate-client` (`openapi-ts`) |
| E2E | `bun run test` → `bunx playwright test` |

## 8. OpenAPI and the generated TypeScript client

The full, verified pipeline lives in [`GENERATED-FILES.md`](GENERATED-FILES.md).
In summary, [`scripts/generate-client.sh`](../../scripts/generate-client.sh):

1. `cd backend`, then `uv run python -c "import app.main; ... app.main.app.openapi()"`
   writes `openapi.json` at the repository root
2. moves it to `frontend/openapi.json`
3. runs `bun run --filter frontend generate-client`, which is
   `@hey-api/openapi-ts` 0.73.0 reading `frontend/openapi-ts.config.ts`
4. runs `bun run lint` (Biome) over the result

Configuration: `input: "./openapi.json"`, `output: "./src/client"`, plugins
`legacy/axios`, `@hey-api/sdk` (`asClass: true`, `classNameBuilder: "{{name}}Service"`),
and `@hey-api/schemas`.

Two facts that the pre-audit draft got wrong:

- **`openapi.json` is git-ignored** (`frontend/.gitignore`) — it is a build
  artifact, not a committed contract file
- **`frontend/src/client/**` and `frontend/src/routeTree.gen.ts` are tracked**
  and committed

Regeneration is enforced automatically: the `generate-frontend-sdk`
pre-commit hook runs `scripts/generate-client.sh` whenever anything under
`backend/` changes.

## 9. Testing

| Layer | Tooling | Present? |
|---|---|---|
| Backend unit + integration | Pytest + `coverage`, FastAPI `TestClient` | **Yes** — 7 test files under `backend/tests/` |
| Frontend component / unit | — | **No runner installed.** No Vitest, no Jest, no Testing Library |
| E2E | Playwright 1.61.1 | **Yes** — 6 specs under `frontend/tests/` |

Backend tests run against a **real PostgreSQL database**, not an in-memory
substitute: `backend/tests/conftest.py` opens a session against the
configured engine, calls `init_db`, and deletes all `Item` and `User` rows
at session teardown. A database must therefore be running for any backend
test to execute.

Playwright runs Chromium only (Firefox/WebKit are commented out), uses a
`setup` project that authenticates once into `playwright/.auth/user.json`,
and boots the dev server via `webServer` at `http://localhost:5173`. In CI
it runs inside the compose `playwright` service, sharded 4 ways.

**`bun run test` runs Playwright, not unit tests.** The absence of a
component-test runner is a real gap against the project's own test
strategy; it is being closed deliberately — see
[`adr/ADR-0007-FRONTEND-TEST-RUNNER.md`](adr/ADR-0007-FRONTEND-TEST-RUNNER.md).

## 10. Docker Compose services

[`compose.yml`](../../compose.yml) (base):

| Service | Image / build | Notes |
|---|---|---|
| `db` | `postgres:18` | healthcheck `pg_isready`, named volume `app-db-data` |
| `adminer` | `adminer` | DB browser, Traefik-labelled |
| `prestart` | backend image | runs `scripts/prestart.sh` (migrate + seed), must complete before `backend` starts |
| `backend` | `backend/Dockerfile` | healthcheck hits `/api/v1/utils/health-check/` |
| `frontend` | `frontend/Dockerfile` | nginx serving the built SPA |

[`compose.override.yml`](../../compose.override.yml) (local development) adds:

| Service | Notes |
|---|---|
| `proxy` | Traefik 3.6, ports 80 and 8090 |
| `mailcatcher` | `schickling/mailcatcher`, UI on 1080, SMTP on 1025 |
| `playwright` | built from `frontend/Dockerfile.playwright`, report port 9323 |

and exposes `db:5432`, `adminer:8080`, `backend:8000`, `frontend:5173`,
switches the backend to `fastapi run --reload` with file-sync watch, and
sets `traefik-public` to `external: false`.

## 11. CI/CD

| Workflow | Trigger | What it does |
|---|---|---|
| `test-backend.yml` | push to `master`, PR | compose up `db` + `mailcatcher`, migrate, run pytest, upload coverage HTML, **`coverage report --fail-under=90`** |
| `playwright.yml` | push to `master`, PR (path-filtered) | `uv sync`, `bun ci`, `generate-client.sh`, compose build, Playwright in 4 shards, merged HTML report |
| `pre-commit.yml` | PR | runs `prek` across the diff and auto-commits formatting fixes |
| `test-docker-compose.yml` | push to `master`, PR | builds the stack and curls backend + frontend |
| `deploy-staging.yml` | push to `master` | **disabled during VS-000** — see below |
| `deploy-production.yml` | GitHub release published | **disabled during VS-000** — see below |
| `guard-dependencies.yml` | PR touching `pyproject.toml`/`uv.lock` | closes PRs from non-org authors |
| `smokeshow`, `labeler`, `issue-manager`, `add-to-project`, `latest-changes`, `bump-pre-commit-hooks`, `detect-conflicts`, `zizmor` | various | upstream-template repository automation |

The **90% coverage gate is a hard constraint on every future slice** and
is not negotiable per-slice; it is CI, not a guideline.

Both deploy workflows target **self-hosted runners** labelled `staging` /
`production` which do not exist for this project, and were gated only on
`github.repository_owner != 'fastapi'` — a condition that now passes on the
personal fork. They have been disabled in VS-000; see
[`adr/ADR-0008-DEPLOYMENT-WORKFLOWS.md`](adr/ADR-0008-DEPLOYMENT-WORKFLOWS.md).

## 12. Pre-commit (prek)

Hooks configured in [`.pre-commit-config.yaml`](../../.pre-commit-config.yaml):
`check-added-large-files`, `check-toml`, `check-yaml`, `end-of-file-fixer`,
`trailing-whitespace` (both excluding `frontend/src/client/`), `typos`,
Biome check, `ruff check --fix`, `ruff format`, `mypy backend/app`,
`ty check backend/app`, **`generate-frontend-sdk`** (runs
`scripts/generate-client.sh` on any `backend/**` change),
`add-release-date`, and `zizmor` over the workflow files.

## 13. Configuration and environment

`Settings` (pydantic-settings) reads `../.env` relative to `backend/`, i.e.
the repository-root `.env`. `SQLALCHEMY_DATABASE_URI` is composed from the
`POSTGRES_*` variables using the `postgresql+psycopg` scheme.
`_enforce_non_default_secrets` **warns** on `changethis` when
`ENVIRONMENT == "local"` and **raises** otherwise.

Current `.env` state (names and status only — values are not reproduced in
documentation): `PROJECT_NAME` is still `"Full Stack FastAPI Project"`,
`STACK_NAME` is `full-stack-fastapi-project`, and **three secrets
(`SECRET_KEY`, `POSTGRES_PASSWORD`, `FIRST_SUPERUSER_PASSWORD`) are still
the literal placeholder `changethis`**. These must be replaced before any
non-local deployment.

## 14. Local toolchain observed on the development machine

| Tool | Version | Note |
|---|---|---|
| `uv` | 0.11.29 | |
| `bun` | 1.3.14 | |
| `node` | 24.11.1 | |
| `git` | 2.52.0.windows.1 | |
| Docker CLI | 29.5.2 | |
| Docker Compose | v5.1.3 | |
| Python (`.venv`) | 3.14.6 | correct for the project |
| Python (system `PATH`) | 3.11.9 | **below** the `>=3.14` requirement — always go through `uv run` |
| Docker daemon | **not running** | `docker info` failed at audit time |

Because the Docker daemon was not running, **no migration, backend test,
E2E run, build or lint has been verified to pass on this machine.** See
[`../delivery/BASELINE-VERIFICATION.md`](../delivery/BASELINE-VERIFICATION.md)
for exactly what was and was not executed.

## 15. Inherited from the template, and its disposition

| Capability | Keep / Replace / Retire | Where decided |
|---|---|---|
| FastAPI + SQLModel + Alembic + PostgreSQL foundation | Keep | ADR-0004 |
| JWT auth, password hashing, recovery/reset flow | Keep, extend with org roles | ADR-0006 |
| `User` model and `/users` routes | Keep, extend | ADR-0004 |
| `is_superuser` platform flag | Keep as-is, do **not** merge with org Owner | ADR-0006 |
| `Item` model, routes, components, tests | Retire at VS-002, once the catalog has equivalent coverage | ADR-0004 |
| Generated client pipeline, `openapi-ts` config | Keep | ADR-0004 |
| shadcn/Radix/Tailwind component library, `DataTable` | Keep and reuse heavily | ADR-0004 |
| Playwright E2E harness and auth setup | Keep | ADR-0004 |
| Compose stack (db, backend, frontend, adminer, mailcatcher, playwright) | Keep | ADR-0004 |
| Traefik + deploy workflows | Retire or reconfigure at VS-019 | ADR-0008 |
| `copier.yml`, `.copier/`, `hooks/post_gen_project.py` | Retire at VS-019 | ADR-0005 |
| `release-notes.md`, `img/`, template `README.md` | Retire/rewrite at VS-019 | ADR-0004 |
| Upstream repo automation workflows | Retire at VS-019 | ADR-0004 |
