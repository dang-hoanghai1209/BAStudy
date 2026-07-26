# Target Architecture

**Status:** Decided (structure and constraints) / Draft (module boundaries
beyond VS-006). Nothing here describes what exists today — for that, read
[`CURRENT-STATE.md`](CURRENT-STATE.md).

## 1. Shape

SellerMargin stays a **modular monolith**: one FastAPI application, one
PostgreSQL database, one React SPA. No microservices, no message broker,
no Kubernetes. Rationale in
[`adr/ADR-0001-MODULAR-MONOLITH.md`](adr/ADR-0001-MODULAR-MONOLITH.md).

The template's technical foundation is kept, not replaced
([`adr/ADR-0004-TEMPLATE-INHERITANCE.md`](adr/ADR-0004-TEMPLATE-INHERITANCE.md)).
What SellerMargin adds is domain: tenancy, catalog, import, costs, fees,
profit, settlement, exceptions, audit.

## 2. Target backend structure

The backend is flat today (`app/models.py`, `app/crud.py`,
`app/api/routes/*.py`). The target is a package-per-domain layout:

```
backend/app/
├── main.py
├── core/            config, db, security  (unchanged)
├── api/
│   ├── main.py      router aggregation
│   ├── deps.py      SessionDep, CurrentUser, OrgScope, RequirePermission
│   └── routes/      thin HTTP layer, one module per domain
└── domains/
    ├── identity/        user, membership, role, permission
    ├── organizations/   organization, shop, marketplace
    ├── catalog/         product, variant, internal SKU, channel listing, mapping
    ├── imports/         import batch, source row, column profile, validation
    ├── orders/          order, order line, status history
    ├── costs/           cost history
    ├── fees/            fee rule, fee type, normalized charges
    ├── profitability/   calculation engine, ledger, explain
    ├── settlements/     statement, statement line, payout
    ├── reconciliation/  matching, variance, period lock
    ├── exceptions/      exception queue and workflow
    ├── reports/         aggregation, export
    └── audit/           audit events
```

Each domain package holds its own `models.py`, `service.py`, `schemas.py`
and `repository.py` (or equivalent). Routes stay thin: validate, delegate
to a service, return a schema. Business rules live in services, never in
route handlers and never in the frontend.

**This restructuring is not a slice by itself and must not be bundled into
a feature slice.** The migration path and its trigger point are recorded in
[`adr/ADR-0009-BACKEND-MODULE-STRUCTURE.md`](adr/ADR-0009-BACKEND-MODULE-STRUCTURE.md).

## 3. Data layering

Input is never written straight into business tables. Four layers, so that
a wrong number can be isolated to a layer:

```
CSV upload / (later) marketplace API
  └── SOURCE LAYER      import_batch, import_source_row        (raw, immutable)
        └── CANONICAL LAYER   column profile, validation, staging
              └── BUSINESS LAYER   order, order_line, internal_sku,
                                   cost_history, fee_rule, refund
                    └── CALCULATED LAYER  profit ledger + input traces
                          └── RECONCILED LAYER  statement, payout, variance
                                └── PRESENTATION  dashboard, drill-down, export
```

Rules that follow from this and are not negotiable per-slice:

- A raw source row is never edited. Corrections happen by re-import or by
  an explicit, audited business-layer change.
- Every business record links back to the source row that produced it.
- Every calculated figure links to its inputs *and to the version of each
  rule used*, not just to the values.
- Aggregates are derived from the calculated layer, never recomputed
  independently in the presentation layer. Dashboard totals and drill-down
  detail must come from the same query path.

## 4. Tenancy

Every business table carries `organization_id`. Tables describing something
shop-specific additionally carry `shop_id`. Access is deny-by-default and
enforced server-side on every request through a shared scoping dependency —
not per-handler, and never only in the UI.

A request for a record outside the caller's organization returns **404, not
403**, so that existence is not leaked.

Decided in [`adr/ADR-0002-TENANCY-MODEL.md`](adr/ADR-0002-TENANCY-MODEL.md);
permissions in [`../security/AUTHORIZATION-POLICY.md`](../security/AUTHORIZATION-POLICY.md).

PostgreSQL row-level security is **not** adopted now. If added later it is
a second layer of defence, never a replacement for application checks.

## 5. Money

Single currency for the MVP, stored explicitly on every monetary record.
All money is `Numeric(18,4)` in PostgreSQL and `decimal.Decimal` in Python —
never `float`, at any layer, including serialization and aggregation.
Rounding is half-up to the currency's minor unit, applied once at a
documented boundary. Full rules and the deferral of multi-currency:
[`adr/ADR-0003-MONEY-AND-CURRENCY.md`](adr/ADR-0003-MONEY-AND-CURRENCY.md).

## 6. Time

Timestamps are stored timezone-aware in UTC (the template already does this
for `created_at`). Business dates that drive period assignment — order date,
cost `effective_from`, settlement date — are governed by
[`HISTORICAL-DATA-POLICY.md`](HISTORICAL-DATA-POLICY.md). The reporting
timezone and fiscal-period definition are still **Open** and block VS-007
and VS-014.

## 7. Request flow

```
React (TanStack Query)
  → generated TypeScript client (axios)
    → FastAPI route: schema validation
      → auth dependency: authenticated? active?
        → org scope dependency: which organization, which role
          → permission check: may this actor perform this action?
            → domain service: business rules
              → repository / SQLModel: PostgreSQL
            ← audit event emitted for auditable actions
      ← response schema
  ← React renders, including loading / empty / error states
```

Nothing skips the org-scope step. A route that does not need it is either
unauthenticated (login, health) or a platform-admin route.

## 8. Asynchronous work

FastAPI `BackgroundTasks` is sufficient for MVP-sized CSV imports. Celery
and Redis are **not** introduced until there is a measured need — a
durable-retry requirement, or import sizes that make in-process handling
unacceptable. Adding them is an ADR, not an implementation detail.

The consequence is accepted explicitly: an in-process background task does
not survive a restart. Import batches therefore carry an explicit status so
an interrupted batch is visibly incomplete rather than silently lost, and
can be reprocessed.

## 9. Query strategy for reporting

Start with plain SQL and PostgreSQL views over the calculated layer. Move
to materialized views only when a measured latency problem justifies the
staleness trade-off. Do not pre-emptively denormalize.

## 10. Frontend

Structure follows the existing template conventions — file-based TanStack
Router routes, TanStack Query for server state, shadcn/Radix components in
`components/ui/`, feature components grouped per domain folder. New feature
folders mirror the backend domains (`components/Catalog/`,
`components/Imports/`, `components/Reconciliation/`, …).

Rules:

- The generated client in `src/client/` is the only way to reach the API.
- Every new screen implements loading, empty and error states — not just
  the happy path.
- No business rule is reimplemented in the frontend. If the UI needs a
  derived number, the API returns it.
- Permission-based hiding is an affordance, never the enforcement.

## 11. Local runtime target (8 GB RAM)

| Component | Approach |
|---|---|
| PostgreSQL | Docker (`compose up -d db`) |
| Backend | Local via `uv run`, not containerised during development |
| Frontend | Local via `bun run dev` |
| Mailcatcher | Only when working on email flows |
| Playwright | Only for E2E runs, not on every change |
| Traefik | Not needed locally |
| Redis / Celery | Not present |

## 12. What is deliberately out of the target

Full ERP or inventory management, fulfilment and shipping labels, HR/CRM/supplier
management, tax filing, double-entry accounting, microservices, Kafka,
Kubernetes, multi-region, and any AI feature that runs ahead of a
trustworthy profit engine. See
[`../ba/01-PRODUCT-VISION.md`](../ba/01-PRODUCT-VISION.md).
