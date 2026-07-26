# Domain and Data Model

**Status:** Verified (what exists) / Decided (conventions) / Draft (target
entities, which firm up per slice).

## 1. What exists today

Two tables, both inherited from the template. **No SellerMargin business
entity exists yet, and there are no money columns anywhere in the schema.**

| Table | Columns |
|---|---|
| `user` | `id` UUID PK, `email` unique indexed, `hashed_password`, `is_active`, `is_superuser`, `full_name`, `created_at` (timezone-aware) |
| `item` | `id` UUID PK, `title`, `description`, `created_at`, `owner_id` FK → `user.id` **ON DELETE CASCADE** |

ORM is **SQLModel**; migrations are **Alembic**, with a linear chain and a
single head, `fe56fa70289e`. All models currently live in one file,
`backend/app/models.py` — the backend is flat, see
[`adr/ADR-0009-BACKEND-MODULE-STRUCTURE.md`](adr/ADR-0009-BACKEND-MODULE-STRUCTURE.md).

`item` is retired at VS-002
([ADR-0004](adr/ADR-0004-TEMPLATE-INHERITANCE.md)). Its `ON DELETE CASCADE`
is a template pattern and must **not** be copied onto SellerMargin business
tables, which deactivate rather than delete.

## 2. Conventions — binding for every new table

| Convention | Rule |
|---|---|
| Tenancy | `organization_id` non-null, indexed, FK. `shop_id` additionally where the record is shop-specific, with a constraint keeping it consistent with the shop's organization ([ADR-0002](adr/ADR-0002-TENANCY-MODEL.md)) |
| Primary keys | UUID, matching the template |
| Money | `NUMERIC(18,4)`, `Decimal` in Python, explicit currency column. **Never `float`** ([ADR-0003](adr/ADR-0003-MONEY-AND-CURRENCY.md)) |
| Timestamps | `DateTime(timezone=True)`, stored UTC |
| Deletion | Deactivate or supersede. No `ON DELETE CASCADE` on business data ([historical-data policy](HISTORICAL-DATA-POLICY.md) §6) |
| Effective dating | `effective_from` inclusive, `effective_to` exclusive and nullable, with a non-overlap constraint |
| Lineage | Every business record references the source row that produced it |
| Attribution | `created_by` / `updated_by` on anything a user changes |
| Placement | Add to `backend/app/models.py`, grouped and section-commented by domain, so the VS-005a split is mechanical |

## 3. Data layering

Input is never written straight into business tables:

```
CSV / (later) marketplace API
  └── SOURCE      import_batch, import_source_row            immutable
        └── CANONICAL   column profile, validation, staging
              └── BUSINESS   order, order_line, internal_sku,
                             cost_history, fee_rule, refund
                    └── CALCULATED   profit ledger + input traces
                          └── RECONCILED   statement, payout, variance
                                └── PRESENTATION   dashboard, drill-down, export
```

## 4. Target entities

Introduced by the slice named. Nothing here exists yet.

| Entity | Purpose | Slice | Scope |
|---|---|---|---|
| `organization` | Tenant boundary; holds the organization currency | VS-001 | — |
| `membership` | User × organization × role | VS-001 | org |
| `shop` | A seller's storefront on one marketplace | VS-001 | org |
| `marketplace` | Reference list | VS-001 | global |
| `audit_event` | Who changed what, when, why | VS-001 | org |
| `product` | Canonical product | VS-002 | org |
| `product_variant` | Variant of a product | VS-002 | org |
| `internal_sku` | SellerMargin's canonical SKU | VS-002 | org |
| `channel_listing` | How a product appears on a channel | VS-003 | org + shop |
| `channel_sku_mapping` | Channel SKU → internal SKU | VS-003 | org |
| `import_batch` | One upload event | VS-004 | org + shop |
| `import_source_row` | Raw row, immutable | VS-004 | org + shop |
| `import_profile` | Saved column mapping per marketplace | VS-004 | org |
| `import_error` | Row-level validation failure | VS-005 | org |
| `cost_history` | Time-versioned COGS per SKU | VS-006 | org |
| `order`, `order_line` | Canonical order data | VS-007 | org + shop |
| `order_status_history` | Lifecycle transitions | VS-007 | org + shop |
| `fee_type`, `fee_rule` | Fee taxonomy and versioned rules | VS-008 | org |
| `order_charge` | Normalized fee/discount/shipping amounts | VS-008 | org + shop |
| `profit_calculation` | Append-only calculation ledger | VS-009 | org + shop |
| `calculation_input_trace` | Which inputs and rule versions were used | VS-009 | org |
| `refund`, `refund_line`, `adjustment` | Reversals | VS-010 | org + shop |
| `statement`, `statement_line`, `payout` | Settlement inputs | VS-011 | org + shop |
| `reconciliation_run`, `_match`, `_variance` | Matching results | VS-012 | org + shop |
| `period_lock` | Closed periods | VS-012 | org + shop |
| `exception` | Unified problem queue | VS-013 | org |
| `role`, `permission`, `role_permission` | Granular RBAC | VS-016 | org |
| `marketplace_account` | Channel credentials | later (API phase) | org + shop |

`user` is extended by membership relationships. **`is_superuser` semantics
are not changed** — it is platform administration and grants no
organization data access
([ADR-0006](adr/ADR-0006-AUTHORIZATION-MODEL.md)).

## 5. Cost history pattern (non-negotiable)

Never model cost as a single overwritable field. Model it as history:

```
cost_history
  organization_id, internal_sku_id
  cost_amount      NUMERIC(18,4)
  currency
  effective_from   inclusive
  effective_to     exclusive, nullable = currently active
  created_by, created_at, note
```

A calculation uses the record whose range **covers the order's sale date**
— not "the current cost" — unless a business rule explicitly says
otherwise (mandatory question #7, still open). Ranges never overlap; at
most one open-ended record per SKU. Corrections append and supersede.

The same pattern applies to `fee_rule`.

## 6. Access control

Deny-by-default, enforced server-side on every request through a shared
scoping dependency — never per-handler and never only in the UI.
Cross-organization access returns **404, not 403**. Full matrix:
[`../security/AUTHORIZATION-POLICY.md`](../security/AUTHORIZATION-POLICY.md).

PostgreSQL row-level security is not adopted. If added later it is defence
in depth, not a replacement for application checks.

## 7. Open items

| Item | Blocks |
|---|---|
| D-08 — timezone and period-boundary policy for business dates | VS-007, VS-014 |
| D-09 — natural key for order and order line; update vs. version on re-import | VS-004, VS-007 |
| D-06 — is SKU remapping retroactive (does `channel_sku_mapping` need effective dating)? | VS-003 |
| D-11 — bundles/kits, which would make mapping 1:N | VS-003 |
| D-13 / P-1 — are personal-data columns stored verbatim, masked, or dropped? | VS-004 |
| Whether RLS is adopted, and when | later |

## 8. Maintenance rule

This file changes in the same pull request as the migration it describes.
A schema that exists without an entry here, or an entry here without a
migration, is a defect.
