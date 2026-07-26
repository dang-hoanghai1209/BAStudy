# CLAUDE.md — SellerMargin Operating Guide

Entry point for Claude Code working in this repository. It stays short.
Detailed rules live in the linked documents — read the relevant one before
starting work in that area, don't rely on memory of it.

> Every fact in this file was verified against the repository during the
> VS-000 audit (2026-07-26). There are no unverified assumptions left. If
> you find something here that contradicts the repository, that is a defect
> — report it (see §8) rather than working around it.

---

## 1. Product mission and boundaries

**SellerMargin** is an E-commerce Profitability & Settlement Reconciliation
platform for multi-channel sellers. It calculates true per-order, per-SKU,
per-shop profit and reconciles expected payout against actual marketplace
payout.

**In scope:** CSV ingestion (marketplace APIs later), SKU normalization
across channels, real profit per order/SKU/shop with full traceability to
source data, expected-vs-actual payout reconciliation, and alerting on
missing data, bad cost data or unreconciled amounts.

**Out of scope:** full ERP/inventory, fulfilment and shipping labels,
HR/CRM/supplier management, tax filing, double-entry accounting,
microservices/Kafka/Kubernetes, and any AI feature that runs ahead of a
trustworthy profit engine.

Full narrative: [`docs/ba/01-PRODUCT-VISION.md`](docs/ba/01-PRODUCT-VISION.md),
[`docs/ba/00-PROBLEM-STATEMENT.md`](docs/ba/00-PROBLEM-STATEMENT.md).

## 2. Source-of-truth hierarchy

When documents conflict, highest wins:

1. An explicit human decision — recorded in an ADR or a merged PR description
2. [`docs/ba/08-BUSINESS-RULES.md`](docs/ba/08-BUSINESS-RULES.md) — business/calculation questions
3. [`docs/architecture/DATA-MODEL.md`](docs/architecture/DATA-MODEL.md) — entity/schema questions
4. [`docs/architecture/API-CONTRACT-POLICY.md`](docs/architecture/API-CONTRACT-POLICY.md) + the generated schema — API shape
5. [`docs/security/AUTHORIZATION-POLICY.md`](docs/security/AUTHORIZATION-POLICY.md) — who may do what
6. [`docs/delivery/VERTICAL-SLICE-ROADMAP.md`](docs/delivery/VERTICAL-SLICE-ROADMAP.md) — sequencing
7. Everything else is supporting detail, not a tie-breaker

**[`docs/architecture/CURRENT-STATE.md`](docs/architecture/CURRENT-STATE.md)
describes only what exists today; nothing may move into it from
`TARGET-ARCHITECTURE.md` without repository evidence.**

If a document is silent or ambiguous, stop and ask rather than inferring a
business rule.

## 3. Decisions already made — do not re-litigate

| Decision | Where |
|---|---|
| Modular monolith; no microservices | [ADR-0001](docs/architecture/adr/ADR-0001-MODULAR-MONOLITH.md) |
| `organization_id` on every business table; `shop_id` where shop-specific | [ADR-0002](docs/architecture/adr/ADR-0002-TENANCY-MODEL.md) |
| Single-currency MVP; `NUMERIC(18,4)` / `Decimal`; never `float` | [ADR-0003](docs/architecture/adr/ADR-0003-MONEY-AND-CURRENCY.md) |
| Template kept; `Item` retired at VS-002, not before | [ADR-0004](docs/architecture/adr/ADR-0004-TEMPLATE-INHERITANCE.md) |
| Upstream tracked via git, not Copier | [ADR-0005](docs/architecture/adr/ADR-0005-UPSTREAM-TRACKING.md) |
| `is_superuser` (platform) and Owner (organization) stay **separate** | [ADR-0006](docs/architecture/adr/ADR-0006-AUTHORIZATION-MODEL.md) |
| Vitest + Testing Library, added in its own commit (VS-000a) | [ADR-0007](docs/architecture/adr/ADR-0007-FRONTEND-TEST-RUNNER.md) |
| Deploy workflows disabled until real infrastructure exists | [ADR-0008](docs/architecture/adr/ADR-0008-DEPLOYMENT-WORKFLOWS.md) |
| Backend stays flat until VS-005a, then domain packages | [ADR-0009](docs/architecture/adr/ADR-0009-BACKEND-MODULE-STRUCTURE.md) |

## 4. Vertical-slice workflow

Every unit of delivery is a vertical slice, not a layer. The order is fixed:

```
Business analysis (rules + acceptance criteria)
  → data model change + reviewed migration
    → API / contract
      → backend implementation
        → regenerate the TypeScript client (never hand-edit)
          → frontend implementation
            → tests (unit, integration, contract, frontend, E2E)
              → UAT note
                → integration report
                  → human review
```

Slice specs: [`docs/delivery/slices/`](docs/delivery/slices/).
Sequencing: [`docs/delivery/VERTICAL-SLICE-ROADMAP.md`](docs/delivery/VERTICAL-SLICE-ROADMAP.md).
Live status: [`docs/delivery/TASK-BOARD.md`](docs/delivery/TASK-BOARD.md).
Reusable prompts: [`docs/prompts/`](docs/prompts/).

## 5. Mandatory gates before coding

Before writing any code for a slice:

- The relevant business rules exist in
  [`docs/ba/08-BUSINESS-RULES.md`](docs/ba/08-BUSINESS-RULES.md) — not
  invented while coding
- Acceptance criteria exist for every story the slice touches
- Every mandatory business question the slice touches is **answered**, not
  assumed
- The slice spec is at **Specified** level, not Seeded
- The Definition of Ready in
  [`docs/delivery/DEFINITION-OF-READY-DONE.md`](docs/delivery/DEFINITION-OF-READY-DONE.md)
  passes

If any is missing, produce the missing artifact first. Do not code around
the gap.

## 6. Git and migration safety

- **The default branch is `master`.** There is no `main`.
- One branch per slice: `slice/VS-0xx-short-name`, off `master`.
- Never commit directly to `master`. Slices land via reviewed PR.
- Alembic autogenerate produces a **candidate**, not a correct migration.
  Read every line against the checklist in
  [`docs/delivery/MIGRATION-AND-ROLLBACK.md`](docs/delivery/MIGRATION-AND-ROLLBACK.md) §3.
  Current head: `fe56fa70289e`.
- Never edit a merged migration. Fix forward.
- Anything with history — cost, fee rule, calculation result — is appended
  and superseded, never overwritten
  ([`docs/architecture/HISTORICAL-DATA-POLICY.md`](docs/architecture/HISTORICAL-DATA-POLICY.md)).
- No destructive schema change without explicit human sign-off recorded in
  the PR description.
- Business tables do **not** use `ON DELETE CASCADE`. The template does for
  `item.owner_id`; do not copy it.

## 7. Generated files — never hand-edited

| File | Regenerated by |
|---|---|
| `frontend/src/client/**` (tracked) | `bash scripts/generate-client.sh` |
| `frontend/src/routeTree.gen.ts` (tracked) | TanStack Router Vite plugin |
| `openapi.json` (**git-ignored**, a build artifact) | FastAPI `app.openapi()` |
| `uv.lock`, `bun.lock` | `uv` / `bun` |

The `generate-frontend-sdk` pre-commit hook runs the generation script on
any `backend/**` change, so the client cannot silently drift.

**Every router must declare `tags=[...]`** — operation IDs are
`{tag}-{name}`, and the first tag becomes the generated service class name.
A missing tag breaks generation; a changed tag is a breaking change.

Details: [`docs/architecture/GENERATED-FILES.md`](docs/architecture/GENERATED-FILES.md).

## 8. Verified commands

```bash
uv sync --all-packages                          # install backend (venv is at repo ROOT)
bun ci                                          # install frontend
docker compose up -d db mailcatcher             # database (required for backend tests)
cd backend && uv run bash scripts/prestart.sh   # alembic upgrade head + seed
cd backend && uv run bash scripts/test.sh       # pytest + coverage
uv run coverage report --fail-under=90          # the real CI gate
cd backend && uv run bash scripts/lint.sh       # mypy (strict), ty, ruff
bash scripts/generate-client.sh                 # regenerate OpenAPI + TS client
bun run dev                                     # Vite on :5173
bun run --filter frontend build
bun run lint                                    # Biome — WRITES files
docker compose run --rm playwright bunx playwright test   # E2E
```

Two traps: the system Python on `PATH` is **3.11.9** and cannot run this
project (`>=3.14` required) — always go through `uv run`; and
**`bun run test` runs Playwright, not unit tests**. There is no
component-test runner until VS-000a lands.

Full detail, including what has and has not been verified to pass:
[`docs/delivery/BASELINE-VERIFICATION.md`](docs/delivery/BASELINE-VERIFICATION.md).

## 9. Testing requirements

Per [`docs/delivery/TEST-STRATEGY.md`](docs/delivery/TEST-STRATEGY.md),
every slice includes: unit tests for every rule or formula it introduces;
integration tests for every endpoint it adds or changes, including the
authorization matrix and a cross-organization 404; a regenerated client;
frontend tests for loading/empty/error states (once VS-000a lands); a
Playwright E2E for the primary happy path; and a UAT note per
[`docs/ba/09-UAT-STRATEGY.md`](docs/ba/09-UAT-STRATEGY.md).

CI enforces **90% backend coverage**. That is a gate, not a target.

A slice without traceable tests for its acceptance criteria is not done,
however well the code runs.

## 10. Stop conditions

Stop and ask the human before proceeding when:

- A business rule the slice needs is unanswered in
  [`docs/ba/08-BUSINESS-RULES.md`](docs/ba/08-BUSINESS-RULES.md)
- A schema change would be destructive or would rewrite calculation history
- The repository contradicts something stated in this document set
- The slice's scope is expanding beyond its spec
- Two source documents conflict and §2 doesn't resolve it
- The work would require a dependency, lockfile, `.env` or Docker change
  that the slice did not explicitly authorise

## 11. Document map

- **Business analysis** — [`docs/ba/`](docs/ba/)
- **Product & UX** — [`docs/product/`](docs/product/)
- **Architecture & ADRs** — [`docs/architecture/`](docs/architecture/)
- **Security** — [`docs/security/`](docs/security/)
- **Delivery & slices** — [`docs/delivery/`](docs/delivery/)
- **Prompt library** — [`docs/prompts/`](docs/prompts/)

Index and reading order: [`docs/README.md`](docs/README.md).
