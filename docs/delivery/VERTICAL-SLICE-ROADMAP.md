# Vertical Slice Roadmap

**Status:** Decided (sequence) / Draft (pacing). This is the sequencing
plan; live status is in [`TASK-BOARD.md`](TASK-BOARD.md) and the specs
themselves are in [`slices/`](slices/).

## 1. Priority table

| Priority | Capability | BA value | Difficulty | Slice |
|---|---|---|---|---|
| P0 | Tenancy and ownership | Foundational | Medium | VS-001 |
| P0 | Internal SKU catalog | Very high | Medium | VS-002 |
| P0 | Channel SKU mapping | Very high | Medium | VS-003 |
| P0 | Import Center | Very high | Medium-Hard | VS-004, VS-005 |
| P0 | Cost History | High | Medium | VS-006 |
| P0 | Canonical orders | Foundational | Medium | VS-007 |
| P0 | Fee Rule Engine | Very high | Hard | VS-008 |
| P0 | Explainable Profit | Very high | Hard | VS-009 |
| P0 | Dashboard + drill-down | High | Medium | VS-014, VS-015 |
| P1 | Refund / Return | High | Medium | VS-010 |
| P1 | Data-quality alerts and exception workflow | High | Medium | VS-013 |
| P1 | Audit + calculation versioning | Very high | Medium | VS-017 (emission from VS-001) |
| P1 | Statement / Payout Reconciliation | Very high | Hard | VS-011, VS-012 |
| P1 | Roles and permissions hardening | High | Medium | VS-016 |
| P1 | Reports and exports | High | Medium | VS-018 |
| P1 | Production hardening | Foundational | Medium | VS-019 |
| P2 | Indirect expense allocation | High | Medium | later |
| P2 | Pricing simulator | Very high | Medium | later |
| P2 | Marketplace connector, scheduled sync | High | Hard | later |
| P3 | Ads profitability, FIFO costing, AI assistant | Medium | Hard | later |

## 2. Sequence

| Slice | Title | Depends on |
|---|---|---|
| VS-000 | Repository audit, baseline checks, documentation adoption | — |
| **VS-000a** | **Frontend test runner (Vitest + Testing Library)** | VS-000 |
| VS-001 | Organization, Shop and Tenant Ownership | VS-000, VS-000a |
| VS-002 | Product Catalog and Internal SKU | VS-001 |
| VS-003 | Marketplace SKU Mapping | VS-002 |
| VS-004 | CSV Import Center (ingest and staging) | VS-001, VS-003 |
| VS-005 | Import Validation and Error Handling | VS-004 |
| **VS-005a** | **Backend domain-package restructure (pure refactor)** | VS-005 |
| VS-006 | Time-Effective Cost History | VS-002, VS-005a |
| VS-007 | Orders and Order Lines (canonical) | VS-005 |
| VS-008 | Fee, Discount, Shipping and Tax Normalization | VS-007 |
| VS-009 | Profit Calculation Engine | VS-006, VS-007, VS-008 |
| VS-010 | Refunds, Returns and Cancellations | VS-009 |
| VS-011 | Settlement / Payout Import | VS-005 |
| VS-012 | Settlement Reconciliation and Matching | VS-009, VS-011 |
| VS-013 | Exception Management | VS-005, VS-012 |
| VS-014 | Profitability Dashboard | VS-009, VS-010, VS-013 |
| VS-015 | SKU and Order Drill-Down / Traceability Explorer | VS-009, VS-014 |
| VS-016 | Roles and Permissions (RBAC hardening) | VS-001, VS-012 |
| VS-017 | Audit Trail (viewer, query, retention) | VS-006, VS-008, VS-016 |
| VS-018 | Reports and Exports | VS-012, VS-014, VS-016, VS-017 |
| VS-019 | Production Hardening and Deployment | all |

## 3. Two slices that are not features

**VS-000a** and **VS-005a** exist so that a tooling change and a structural
refactor do not ride along inside a feature slice, where they would make
review harder and blur what a slice actually delivered. Both are scheduled
deliberately; neither is scope creep.

VS-000a must land before VS-001 implementation — until it does, no slice
can satisfy the frontend-test line of the Definition of Done.

## 4. Changes from the pre-audit draft

The original draft compressed import into one slice and placed cost history
at VS-004. The audit changed this:

| Draft | Now | Why |
|---|---|---|
| VS-002 catalog + mapping together | VS-002 + VS-003 | Mapping has its own rules, UI and audit surface; bundled, the slice is not reviewable in one pass |
| VS-003 CSV Import Center (everything) | VS-004 + VS-005 | Upload, encoding detection, column mapping, validation, partial import, idempotency and reprocessing is far too much for one PR |
| VS-004 Cost History | VS-006 | Provable only once orders exist; still lands before the profit engine |
| "Planned" bucket | VS-007 – VS-019 | The draft never expanded these into slices with acceptance criteria |
| — | VS-000a, VS-005a | Newly identified; see §3 |

## 5. Why this order

- **Tenancy first** because every later table carries `organization_id`,
  and retrofitting it is a full-schema migration.
- **Catalog and mapping before import** because imported rows need
  somewhere to resolve their SKUs to.
- **Orders before fees before profit** because each is the input to the
  next, and a profit engine built on unnormalized fee data cannot be
  trusted or explained.
- **Cost history before profit** because time-of-sale cost resolution is
  the input most likely to be got wrong, and it is the project's flagship
  correctness claim.
- **Dashboard after profit and refunds**, never before — shipping a
  dashboard on an unproven formula produces confident wrong numbers, which
  is the exact failure this product exists to prevent (R-21).
- **Reconciliation late** because it needs both a trustworthy expected
  figure (VS-009) and real settlement data (VS-011).
- **RBAC and audit query late, but emission early.** The full permission
  matrix needs the complete set of sensitive actions to exist first; audit
  *events*, by contrast, must be emitted from VS-001 onward or they become
  an expensive back-fill (R-17).

## 6. Pacing

Roughly: discovery and foundation (VS-000 – VS-003), trusted data
(VS-004 – VS-007), the profit engine (VS-008 – VS-010), reconciliation and
controls (VS-011 – VS-013), presentation (VS-014 – VS-015), hardening
(VS-016 – VS-019).

Pacing depends on the owner's availability, not a fixed deadline. The
sequence is the commitment; the calendar is not.

## 7. Demo data

The Olist Brazilian E-commerce Dataset (CC BY-NC-SA 4.0) may be used to
exercise order and order-item structures. It has **no COGS, no fee ledger
and no settlement statements**, so it cannot substitute for the real sample
files that VS-004, VS-008 and VS-011 need (blocker B-5). Anything derived
from it where cost or fee data was fabricated must be labelled
**synthetic** — never presented as real or reconciled figures.
