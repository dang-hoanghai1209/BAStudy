# Slice Specifications

One file per vertical slice. Each is created from
[`../VERTICAL-SLICE-SPEC-TEMPLATE.md`](../VERTICAL-SLICE-SPEC-TEMPLATE.md)
**before** any code is written for that slice.

Live status is in [`../TASK-BOARD.md`](../TASK-BOARD.md). Sequencing
rationale is in [`../VERTICAL-SLICE-ROADMAP.md`](../VERTICAL-SLICE-ROADMAP.md).
This folder holds the specs themselves.

## Two levels of completeness

| Level | Which slices | Meaning |
|---|---|---|
| **Specified** | VS-001 – VS-006 | Detailed enough to start once its Definition of Ready passes |
| **Seeded** | VS-007 – VS-019 | Carries objective, entities, acceptance criteria, dependencies, risks and exclusions from the roadmap, with business-rule sections explicitly marked as pending |

A seeded spec is deliberately incomplete. Filling in the business analysis
of VS-012 today would mean inventing reconciliation rules eleven slices
before anyone can validate them — which is precisely the failure mode
R-01 exists to prevent. A seeded spec must be brought to Specified before
its slice can pass the Definition of Ready.

## Index

| Slice | Name | Level | Depends on |
|---|---|---|---|
| VS-000 | Repository audit, baseline checks, documentation adoption | Done (this commit) | — |
| VS-000a | Frontend test runner (Vitest + Testing Library) | Specified — see [ADR-0007](../../architecture/adr/ADR-0007-FRONTEND-TEST-RUNNER.md) | VS-000 |
| [VS-001](VS-001-ORGANIZATION-AND-SHOP.md) | Organization, Shop and Tenant Ownership | Specified | VS-000, VS-000a |
| [VS-002](VS-002-PRODUCT-CATALOG.md) | Product Catalog and Internal SKU | Specified | VS-001 |
| [VS-003](VS-003-SKU-MAPPING.md) | Marketplace SKU Mapping | Specified | VS-002 |
| [VS-004](VS-004-CSV-IMPORT-CENTER.md) | CSV Import Center (ingest and staging) | Specified | VS-001, VS-003 |
| [VS-005](VS-005-IMPORT-VALIDATION.md) | Import Validation and Error Handling | Specified | VS-004 |
| VS-005a | Backend domain-package restructure | Specified — see [ADR-0009](../../architecture/adr/ADR-0009-BACKEND-MODULE-STRUCTURE.md) | VS-005 |
| [VS-006](VS-006-COST-HISTORY.md) | Time-Effective Cost History | Specified | VS-002, VS-005a |
| [VS-007](VS-007-ORDERS.md) | Orders and Order Lines (canonical) | Seeded | VS-005 |
| [VS-008](VS-008-FEE-NORMALIZATION.md) | Fee, Discount, Shipping and Tax Normalization | Seeded | VS-007 |
| [VS-009](VS-009-PROFIT-ENGINE.md) | Profit Calculation Engine | Seeded | VS-006, VS-007, VS-008 |
| [VS-010](VS-010-REFUNDS-AND-RETURNS.md) | Refunds, Returns and Cancellations | Seeded | VS-009 |
| [VS-011](VS-011-SETTLEMENT-IMPORT.md) | Settlement / Payout Import | Seeded | VS-005 |
| [VS-012](VS-012-RECONCILIATION.md) | Settlement Reconciliation and Matching | Seeded | VS-009, VS-011 |
| [VS-013](VS-013-EXCEPTION-MANAGEMENT.md) | Exception Management | Seeded | VS-005, VS-012 |
| [VS-014](VS-014-PROFITABILITY-DASHBOARD.md) | Profitability Dashboard | Seeded | VS-009, VS-010 |
| [VS-015](VS-015-DRILL-DOWN.md) | SKU and Order Drill-Down / Traceability Explorer | Seeded | VS-009, VS-014 |
| [VS-016](VS-016-ROLES-AND-PERMISSIONS.md) | Roles and Permissions (RBAC hardening) | Seeded | VS-001, VS-012 |
| [VS-017](VS-017-AUDIT-TRAIL.md) | Audit Trail | Seeded | VS-006, VS-008, VS-016 |
| [VS-018](VS-018-REPORTS-AND-EXPORTS.md) | Reports and Exports | Seeded | VS-014, VS-012, VS-017 |
| [VS-019](VS-019-PRODUCTION-HARDENING.md) | Production Hardening and Deployment | Seeded | all |

## Two slices that are not features

**VS-000a — frontend test runner.** A dependency change, deliberately kept
out of VS-000 (documentation-only) and out of VS-001 (so a feature slice
does not carry an unrelated tooling diff). It must land before VS-001
implementation, because until it does, no slice can satisfy the
"frontend test for new UI states" line of the Definition of Done.

**VS-005a — backend restructure.** A pure refactor from the flat
`models.py` / `crud.py` layout into domain packages, scheduled between
VS-005 and VS-006 when five real domains exist to draw boundaries from. No
behaviour change, no schema change, no API change — the regenerated client
diff must be empty.

Neither is scope creep; both are scheduled precisely so they do not become
scope creep inside a feature slice.

## Before starting any slice

1. Its spec here is at **Specified** level.
2. The Definition of Ready in
   [`../DEFINITION-OF-READY-DONE.md`](../DEFINITION-OF-READY-DONE.md) passes.
3. Every business question the slice depends on is **answered** in
   [`../../ba/08-BUSINESS-RULES.md`](../../ba/08-BUSINESS-RULES.md) — not
   pending, not assumed.
4. Its dependencies are complete, not merely started.
