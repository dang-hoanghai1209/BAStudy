# VS-019 — Production Hardening and Deployment

**Slice ID:** VS-019 · **Depends on:** all prior slices · **Status:** Not started · **Level:** Seeded

> **Seeded spec.** In practice this is iterated across the project rather
> than done once at the end, but it owns a final checklist.

## Business objective

Make the system safe and operable for real data, and deployable on a real
domain.

## BA learning outcome

Treating **non-functional requirements as business requirements**:
availability, backup and restore, data retention, incident response. These
are commitments to the seller, not infrastructure trivia.

## Business rules required — **pending**

| Question | Status |
|---|---|
| **P-2 / P-3 / P-7** — retention for uploaded files, source rows, audit events | **Open** |
| **P-6** — how a buyer deletion request is handled | **Open** |
| Backup frequency, acceptable data loss (RPO) and downtime (RTO) | **Open** |
| Incident and rollback authority | **Open** |
| **R-12 / ADR-0005** — re-enable, replace or delete the deploy workflows; keep or cut upstream tracking | **Open** |

## Entities involved

None new. Configuration, operations and policy.

## Expected API capabilities

Readiness and liveness endpoints beyond the existing
`GET /api/v1/utils/health-check/`. Rate limiting on authentication
endpoints. Structured request logging with correlation IDs matching the
audit `request_id`. Error reporting through the Sentry hook already wired
in `backend/app/main.py` (active only when `ENVIRONMENT != "local"` and a
DSN is set).

## Expected frontend screens

Production error boundary and 500 page (`react-error-boundary` is already a
dependency); maintenance state; error messaging that does not leak
internals.

## Acceptance criteria

1. All three `changethis` placeholders in `.env` are replaced, and the
   non-local secret guard in `backend/app/core/config.py` passes for the
   target environment.
2. Backup **and a rehearsed restore** are documented — a restore that has
   been executed, not a backup script that has never been tested.
3. A rollback procedure exists covering both application and migration,
   per [`../MIGRATION-AND-ROLLBACK.md`](../MIGRATION-AND-ROLLBACK.md) §5.
4. `PROJECT_NAME`, `STACK_NAME`, `DOMAIN`, the root `package.json` name and
   all page titles carry SellerMargin branding — no "FastAPI Template"
   strings remain.
5. CI is green, including the 90% coverage gate.
6. The ADR-0004 retirement checklist is complete (see below).
7. Deploy workflows are either properly configured and re-enabled, or
   deleted with a recorded decision — not left disabled indefinitely.
8. A security review of authentication, tenancy, export and audit paths has
   been completed and its findings addressed.

## Retirement checklist (from ADR-0004)

- [ ] `copier.yml`, `.copier/`, `hooks/post_gen_project.py`
- [ ] `release-notes.md`, `img/`
- [ ] `README.md`, `CONTRIBUTING.md` rewritten for SellerMargin
- [ ] `development.md`, `deployment.md` rewritten (content is useful — do not simply delete)
- [ ] Upstream automation workflows: `latest-changes`, `add-to-project`,
      `issue-manager`, `labeler`, `bump-pre-commit-hooks`,
      `guard-dependencies`, `smokeshow`, `detect-conflicts`
- [ ] Confirm `Item` and all its artifacts were retired at VS-002
- [ ] Branding strings across backend, frontend and `.env`
- [ ] Upstream-tracking decision confirmed or superseded (ADR-0005)

## Test layers

Integration (rate limiting, readiness/liveness); E2E smoke against a
deployed staging stack; **a restore drill actually executed and
documented**; a security review of auth, tenancy, export and audit paths.

## Dependencies

All prior slices.

## Risks

| Risk | Mitigation |
|---|---|
| **R-12** — deploy workflows firing against nonexistent runners | Already disabled in VS-000 ([ADR-0008](../../architecture/adr/ADR-0008-DEPLOYMENT-WORKFLOWS.md)); this slice resolves them permanently |
| A backup that has never been restored is an assumption, not a safeguard | The restore drill is an acceptance criterion, not a task |
| Hardening deferred entirely to the end and then rushed | Treat as iterative; revisit at each phase boundary |
| Placeholder secrets reaching a real environment | The config guard raises for non-local environments; verified in criterion 1 |

## CV evidence

A production-readiness checklist, a rehearsed restore, a security review,
and a real deployment — evidence most portfolio projects lack entirely.

## Explicit exclusions

No Kubernetes. No autoscaling. No multi-region. No SOC 2 or ISO compliance
programme. No paid monitoring stack beyond Sentry. No CDN or edge caching.
