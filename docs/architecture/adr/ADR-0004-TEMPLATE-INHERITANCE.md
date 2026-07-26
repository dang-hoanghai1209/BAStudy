# ADR-0004 — What is kept, replaced and retired from the Full Stack FastAPI Template

**Status:** Accepted
**Date:** 2026-07-26
**Deciders:** Project owner (human), recorded at VS-000 approval

## Context

The VS-000 audit established that this repository is a **fork of the Full
Stack FastAPI Template repository itself** — 1392 upstream commits,
`copier.yml` present, no rendered `.copier-answers.yml`, root package still
named `fastapi-full-stack-template`, and no file mentioning SellerMargin.
Details in [`../CURRENT-STATE.md`](../CURRENT-STATE.md).

That means the repository carries three distinct kinds of content:

1. **Product foundation** worth keeping (FastAPI/SQLModel/Alembic, auth,
   the generated-client pipeline, the component library, the test harness).
2. **Demo content** that exists to show the template works (`Item`).
3. **Template-maintenance machinery** aimed at the upstream project's own
   maintainers (`copier.yml`, release notes, repo-automation workflows).

Deleting all three at once at the start would destroy working test coverage
and the CI baseline for no product benefit. Keeping all three indefinitely
leaves a portfolio project that looks like an unmodified template.

## Decision

**Keep the foundation, retire the demo content on a schedule tied to
replacement coverage, and retire the template machinery at production
hardening.**

### Keep

| Item | Reason |
|---|---|
| FastAPI + SQLModel + Alembic + PostgreSQL | Exactly the stack the project wants; rebuilding it adds nothing |
| JWT auth, pwdlib hashing, recovery/reset flow, enumeration-safe responses | Correct and non-trivial; extended, not replaced (see ADR-0006) |
| `User` model and `/users` routes | Extended with membership and roles in VS-001 |
| `is_superuser` platform flag | Kept as-is, deliberately **not** merged with organization Owner (ADR-0006) |
| `scripts/generate-client.sh` + `openapi-ts` config | Verified working; the contract discipline it enforces is wanted |
| shadcn/Radix/Tailwind component library, `DataTable`, `Pending*` skeletons | Substantial reusable UI; every SellerMargin screen builds on it |
| Playwright harness, auth `setup` project, mailcatcher integration | Working E2E foundation |
| Compose stack, `prestart` migrate-and-seed pattern | Working local and CI runtime |
| `test-backend.yml`, `playwright.yml`, `pre-commit.yml`, `test-docker-compose.yml` | Real CI value, including the 90% coverage gate |
| Biome, ruff, mypy (strict), ty, prek hooks | Working quality gates |

### Retire on a schedule

| Item | When | Condition |
|---|---|---|
| `Item` model, `/items` routes, `components/Items/*`, `test_items.py`, `items.spec.ts`, `crud.create_item`, `tests/utils/item.py` | **VS-002** | Only once the product/SKU catalog has equivalent test coverage, in **one dedicated commit**. Not touched in VS-000 or VS-001 |
| `copier.yml`, `.copier/`, `hooks/post_gen_project.py` | VS-019 | Template-generation machinery; `copier update` was never viable here (ADR-0005) |
| `release-notes.md` (128 KB of upstream history), `img/` | VS-019 | Upstream artefacts |
| `README.md`, `CONTRIBUTING.md`, `development.md`, `deployment.md` | VS-019 | **Rewrite, not delete** — `development.md` and `deployment.md` contain accurate operational steps worth preserving in SellerMargin form |
| `latest-changes.yml`, `add-to-project.yml`, `issue-manager.yml`, `labeler.yml`, `bump-pre-commit-hooks.yml`, `guard-dependencies.yml`, `smokeshow.yml`, `detect-conflicts.yml` | VS-019 | Upstream-repository automation. `guard-dependencies.yml` in particular closes pull requests from non-org authors, which is actively wrong here |
| `deploy-staging.yml`, `deploy-production.yml` | **Disabled in VS-000**, decided at VS-019 | See ADR-0008 |
| Template branding (`PROJECT_NAME`, `STACK_NAME`, page titles, root package name) | VS-019 | Cosmetic but necessary before anything is shown to a reviewer |

### Replace

| Item | Replaced by |
|---|---|
| Placeholder dashboard (`_layout/index.tsx`, a greeting only) | Profitability dashboard, VS-014 |
| Ad-hoc `owner_id` resource checks in route handlers | Shared organization-scoping dependency, VS-001 |
| Hard delete with `ON DELETE CASCADE` on business data | Deactivation / supersession per [`../HISTORICAL-DATA-POLICY.md`](../HISTORICAL-DATA-POLICY.md) |
| Flat `models.py` / `crud.py` | Domain packages, per ADR-0009, as a standalone refactor |

## Alternatives considered

- **Strip the template to a skeleton immediately** — rejected. It would
  delete the only tests that exist, drop coverage below the 90% CI gate,
  and spend a slice's effort on deletion before any business value exists.
- **Keep `Item` permanently as a harmless example** — rejected. It is
  reachable demo functionality in a portfolio project; a reviewer opening
  the app and finding "Items" learns that the template was not really
  taken over.
- **Regenerate a clean project from the template with Copier and port work
  across** — rejected. There are no Copier answers to regenerate from, and
  it would discard the git history that ADR-0005 relies on for upstream
  tracking.

## Consequences

- Positive: CI stays green throughout; coverage never dips because of
  deletion.
- Positive: each retirement is a small reviewable commit with a clear
  trigger, rather than one large risky sweep.
- Trade-off: template branding and demo content remain visible for several
  slices. Accepted; it is documented rather than hidden.
- Obligation: VS-002 must not be considered done until the `Item` retirement
  commit has landed and coverage still passes.
- Obligation: VS-019 owns a checklist of every remaining retirement item.

## Related

[`ADR-0005-UPSTREAM-TRACKING.md`](ADR-0005-UPSTREAM-TRACKING.md),
[`ADR-0006-AUTHORIZATION-MODEL.md`](ADR-0006-AUTHORIZATION-MODEL.md),
[`ADR-0008-DEPLOYMENT-WORKFLOWS.md`](ADR-0008-DEPLOYMENT-WORKFLOWS.md),
[`ADR-0009-BACKEND-MODULE-STRUCTURE.md`](ADR-0009-BACKEND-MODULE-STRUCTURE.md),
[`../CURRENT-STATE.md`](../CURRENT-STATE.md) §15.
