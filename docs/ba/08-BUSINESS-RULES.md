# Business Rules Catalogue

**Status:** Open — this file is the **source of truth for business logic**
per [`../../CLAUDE.md`](../../CLAUDE.md) §2, but it still contains mostly
questions rather than answers.

A slice must not implement logic covered by an unanswered question below.
Answer it here first — with the human owner, not by assumption — then
implement.

## 1. Mandatory business questions

Each must be answered and recorded in §3 before the code that depends on it
is written.

| # | Question | Blocks |
|---|---|---|
| 1 | Is a discount borne by the seller or the marketplace? | VS-008 |
| 2 | Is fee calculated on gross sales or net sales? | VS-008 |
| 3 | Is fee rounded per item or per order? | VS-008 |
| 4 | When is a refund recognized — order date, refund date, or statement date? | VS-010 |
| 5 | Does a refund also return the commission the marketplace charged? | VS-010 |
| 6 | Is returned merchandise re-added to inventory? | VS-010 (likely N/A — no inventory scope) |
| 7 | Is COGS taken at time-of-sale price or current price? | VS-006, VS-009 |
| 8 | When an old cost is edited, does it trigger recalculation of historical orders? | VS-006, VS-009 |
| 9 | Can a reconciled period be locked against further change? | VS-012 |
| 10 | How is an adjustment with no Order ID allocated? | VS-011, VS-012 |
| 11 | Is a shared/indirect expense allocated by revenue, by units, or manually? | VS-009 |
| 12 | Does a payout belong to the transaction date, the statement date, or the bank-receipt date? | VS-011 |
| 13 | Can one order span multiple payouts? | VS-011, VS-012 |
| 14 | How are late-arriving transactions handled? | VS-010, VS-012 |
| 15 | Who is authorized to edit a fee rule or confirm a reconciliation? | VS-008, VS-012, VS-016 |

## 2. Additional decisions identified by the VS-000 audit

The original list of fifteen did not cover these. Four have since been
decided; the rest are open.

| ID | Decision | Status | Blocks |
|---|---|---|---|
| D-01 | Tenant boundary: organization, shop, or both | **Decided** — `organization_id` everywhere, `shop_id` where shop-specific ([ADR-0002](../architecture/adr/ADR-0002-TENANCY-MODEL.md)) | — |
| D-02 | Single- or multi-currency | **Decided** — single-currency MVP, currency stored explicitly, multi-currency deferred ([ADR-0003](../architecture/adr/ADR-0003-MONEY-AND-CURRENCY.md)) | — |
| D-03 | Money storage type, precision and rounding | **Decided** — `NUMERIC(18,4)` / `Decimal`, half-up, rounded once at a documented boundary ([ADR-0003](../architecture/adr/ADR-0003-MONEY-AND-CURRENCY.md)) | — |
| D-04 | Relationship between platform `is_superuser` and organization Owner | **Decided** — kept separate; a superuser gets no organization data access ([ADR-0006](../architecture/adr/ADR-0006-AUTHORIZATION-MODEL.md)) | — |
| D-05 | Keep, repurpose or retire the template's `Item` | **Decided** — retire at VS-002, once catalog coverage replaces it, in its own commit ([ADR-0004](../architecture/adr/ADR-0004-TEMPLATE-INHERITANCE.md)) | — |
| D-06 | Is SKU remapping retroactive to already-imported orders? | **Open** | VS-003 |
| D-07 | Which marketplaces are in MVP scope, and can real sample exports be obtained? | **Open** | VS-004, VS-008, VS-011 |
| D-08 | Timezone and period-boundary policy: storage UTC; reporting zone; week start; fiscal calendar | **Open** | VS-007, VS-014 |
| D-09 | Order/line natural key per marketplace; update vs. version on re-import | **Open** | VS-004, VS-007 |
| D-10 | Soft delete vs. hard delete for business records | **Decided** — deactivate or supersede; no `ON DELETE CASCADE` on business data ([historical-data policy](../architecture/HISTORICAL-DATA-POLICY.md) §6) | — |
| D-11 | Are bundles/kits in scope (one channel SKU → several internal SKUs)? | **Open** | VS-003 |
| D-12 | Recalculation trigger policy, and who may trigger it | **Open** | VS-009 |
| D-13 | Personal-data policy for raw import data | **Open** — P-1 to P-7 in [`../security/DATA-PRIVACY.md`](../security/DATA-PRIVACY.md) | VS-004, VS-015, VS-018, VS-019 |

## 3. Answered rules

*(None yet. This section fills in as questions above are resolved.)*

Format to use:

```
### BR-001 — <short title>
Statement: <the rule, stated unambiguously and testably>
Rationale: <why>
Decided: <date> by <who>
Affects: <entities, calculations, slices>
Test: <the test that proves it>
```

"Statement" must be testable. *"COGS should be accurate"* is not a rule;
*"COGS is the cost record whose effective range covers the order date"* is.

## 4. Constraints already binding (not numbered rules)

These follow from decisions already made and apply without further debate:

- Every business table carries `organization_id`; a user in one
  organization must never read another's data, even by guessing an
  identifier. Deny-by-default, enforced server-side, **404 not 403**.
- A platform superuser without a membership has **no** access to
  organization business data.
- Money is `NUMERIC(18,4)` / `Decimal` everywhere. Never `float`.
- A row whose currency differs from the organization's currency is
  rejected at import, never coerced.
- Cost and fee-rule changes never silently rewrite historical
  calculations — append and supersede
  ([historical-data policy](../architecture/HISTORICAL-DATA-POLICY.md)).
- Import must be idempotent at both file and row level.
- SKU mapping may suggest but never auto-merge; a human confirms.
- The calculation ledger is append-only; recalculation adds an entry.
- Every auditable action emits exactly one audit event
  ([`../security/AUDIT-POLICY.md`](../security/AUDIT-POLICY.md)).

## 5. Import validation rules

Skeleton — codes and severities are defined in VS-005. Known categories:
missing Order ID; invalid date; a negative amount where the business
disallows one; unsupported or mismatched currency; duplicate Order ID +
Order Item ID; unmapped SKU; order missing COGS; unrecognized fee label;
row missing a required column.

Each needs a stable code, a severity (blocking vs. warning), and a message
written for an operator rather than a developer.

## 6. Maintenance rule

This file changes in the same pull request as any code that depends on it.
A rule that exists only in code is a defect — it is exactly the failure
mode risk R-01 exists to prevent, and the periodic audit
([`../prompts/09-REPO-AUDIT.md`](../prompts/09-REPO-AUDIT.md), item 6)
checks for it.
