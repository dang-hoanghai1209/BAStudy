# Baseline Verification

**Status:** Verified for what was run; **incomplete** for what was not.
**Date of audit:** 2026-07-26 (VS-000)

> **Second attempt, 2026-07-26 (later the same day):** a full baseline run
> was authorised and attempted. `docker info` still failed — diagnostics
> confirmed no Docker Desktop process, no `docker*` named pipe, and
> `com.docker.service` in state **Stopped** (context `desktop-linux`,
> endpoint `npipe:////./pipe/dockerDesktopLinuxEngine`). Per instruction,
> no partial baseline was attempted. §5 remains entirely unexecuted.

This document records the verified command set for this repository, and —
just as importantly — states plainly what has **not** been proven to work.

## 1. Honest summary

> **No test suite, lint, type-check, migration or build has been verified
> to pass on this machine.** The Docker daemon was not running at audit
> time, and backend tests require a live PostgreSQL database. What was
> verified is the presence and version of the toolchain, the structure of
> the repository, and the shape of the CI configuration.

This matters because the pre-audit plan proposed declaring VS-001 "ready"
after an audit. A green baseline is a precondition for that, and it has not
been demonstrated. See §5.

## 2. Environment observed (2026-07-26)

| Tool | Version | Status |
|---|---|---|
| `uv` | 0.11.29 | ✅ |
| `bun` | 1.3.14 | ✅ |
| `node` | 24.11.1 | ✅ |
| `git` | 2.52.0.windows.1 | ✅ |
| Docker CLI | 29.5.2 | ✅ present |
| Docker Compose | v5.1.3 | ✅ present |
| Python — `.venv` | 3.14.6 | ✅ correct |
| Python — system `PATH` | 3.11.9 | ⚠️ **below** the `>=3.14` requirement |
| Docker **daemon** | — | ❌ **not running** |

Two consequences worth remembering:

- **Always invoke Python through `uv run`.** The system interpreter on
  `PATH` is 3.11.9 and cannot run this project. The workspace virtualenv is
  at the **repository root** (`.venv/`), not `backend/.venv/`.
- **Nothing that touches the database can run without starting Docker
  Desktop first.**

## 3. Commands executed during the audit

All read-only. None modified the working tree.

| Command | Result |
|---|---|
| `git rev-parse --abbrev-ref HEAD` | `docs/VS-000-master-plan` |
| `git status --porcelain` | only untracked documentation files |
| `git log --oneline master..HEAD` | empty — branch has no commits ahead of `master` |
| `git rev-list --count HEAD` | 1392 (all upstream template history) |
| `git ls-files frontend/src/client` | 10 generated client files, tracked |
| `git check-ignore` / `frontend/.gitignore` | `openapi.json` is **ignored** |
| `uv --version`, `bun --version`, `node --version`, `git --version` | as tabled above |
| `docker --version`, `docker compose version` | CLI present |
| `docker info` | ❌ failed — daemon not running |
| `.venv/Scripts/python.exe --version` | 3.14.6 |
| Alembic revision-chain inspection (file read) | linear, single head `fe56fa70289e` |
| `.env` placeholder scan (names and counts only) | 3 secrets still `changethis` |

Secret **values** were never read or reproduced.

## 4. The verified command set

These are the correct commands for this repository. Those marked
**unexecuted** are believed correct from configuration but have not been
run here.

### Setup

```bash
uv sync --all-packages          # unexecuted — installs backend deps at repo root .venv
bun ci                          # unexecuted — installs frontend deps
```

### Database

```bash
docker compose up -d db mailcatcher          # unexecuted — daemon down
cd backend && uv run bash scripts/prestart.sh # unexecuted — waits for DB, alembic upgrade head, seed
```

### Backend

```bash
cd backend && uv run bash scripts/lint.sh    # unexecuted — mypy (strict), ty, ruff check, ruff format --check
cd backend && uv run bash scripts/format.sh  # unexecuted — WRITES files
cd backend && uv run bash scripts/test.sh    # unexecuted — coverage run -m pytest tests/, needs a live DB
uv run coverage report --fail-under=90       # unexecuted — the actual CI gate
```

### Frontend

```bash
bun run dev                                  # unexecuted — Vite on :5173
bun run --filter frontend build              # unexecuted — tsc + vite build
bun run lint                                 # unexecuted — Biome, WRITES files
```

### Contract

```bash
bash scripts/generate-client.sh              # unexecuted — WRITES openapi.json and frontend/src/client/
```

Requires `uv sync` and `bun ci` first. Does **not** require a running
server, but does require `.env` to be loadable, because it imports
`app.main` (which constructs `Settings`).

### End to end

```bash
docker compose build                                          # unexecuted
docker compose up -d --wait backend frontend adminer          # unexecuted
curl http://localhost:8000/api/v1/utils/health-check           # unexecuted
docker compose run --rm playwright bunx playwright test        # unexecuted
bash scripts/test.sh                                          # unexecuted — full compose test cycle
```

### Everything, as CI runs it

```bash
uv run prek run --all-files                  # unexecuted — includes the client-regeneration hook
```

## 5. What must be run before VS-001 is genuinely Ready

In order. **This has not been done.**

```bash
# 1. Start Docker Desktop, then:
docker compose up -d db mailcatcher

# 2. Install
uv sync --all-packages
bun ci

# 3. Migrate and seed
cd backend && uv run bash scripts/prestart.sh && cd ..

# 4. Backend tests + the real coverage gate
cd backend && uv run bash scripts/test.sh && uv run coverage report --fail-under=90 && cd ..

# 5. Static analysis
cd backend && uv run bash scripts/lint.sh && cd ..

# 6. Contract regeneration produces no diff
bash scripts/generate-client.sh && git diff --exit-code frontend/src/client/

# 7. Frontend builds
bun run --filter frontend build

# 8. E2E
docker compose build && docker compose run --rm playwright bunx playwright test
```

Record the actual result of each — including failures — in the VS-000
integration report. A step that fails is information, not a reason to skip
reporting it.

## 6. Known constraints these commands operate under

- **Coverage gate: 90%.** `test-backend.yml` runs
  `coverage report --fail-under=90`. Every slice must land near-fully
  covered, and retiring `Item` (ADR-0004) removes covered code — which is
  why that retirement waits for equivalent catalog coverage.
- **Backend tests need a real database.** `backend/tests/conftest.py` opens
  a session against the configured engine, calls `init_db`, and deletes all
  `Item` and `User` rows at session teardown. There is no in-memory
  substitute, and the teardown is destructive — never point tests at a
  database holding anything you want to keep.
- **`bun run test` runs Playwright**, not unit tests. There is no
  component-test runner until VS-000a
  ([`../architecture/adr/ADR-0007-FRONTEND-TEST-RUNNER.md`](../architecture/adr/ADR-0007-FRONTEND-TEST-RUNNER.md)).
- **Biome and ruff format commands write files.** Do not run them when the
  task forbids modifying the working tree.
- **The `generate-frontend-sdk` pre-commit hook** runs
  `scripts/generate-client.sh` on any `backend/**` change, so a commit
  touching the backend regenerates the client automatically.
- **Deploy workflows are disabled** (ADR-0008); pushing to `master` will
  not trigger a deployment.
