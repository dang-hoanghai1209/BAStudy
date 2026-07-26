# UX Principles

**Status:** Decided (principles) / Draft (specific patterns, which firm up
as screens are built).

SellerMargin's product promise is *trust in a number*. These principles all
serve that promise. Where a principle conflicts with visual polish, the
principle wins.

## 1. Never present an untrustworthy number as trustworthy

The most damaging thing this product can do is show a confident figure
built on incomplete data.

- If inputs are missing (unmapped SKUs, missing costs, unprocessed batches),
  say so **next to the number**, not on a separate page.
- Distinguish **zero** from **no data** from **not yet calculated**. Three
  different states, three different displays, never the same "0".
- Partial data gets a coverage indicator ("based on 812 of 1,004 orders").
- A figure whose inputs changed since it was calculated is marked stale.

## 2. Every number is one click from its explanation

Traceability is a feature, not a database property.

- Any displayed figure exposes: what it is made of, which records produced
  it, and which rule version was applied.
- Aggregate and detail come from the same query path. If drill-down does
  not sum to the headline, that is a defect, not a rounding remark.
- The explanation uses business vocabulary from
  [`../ba/03-DOMAIN-GLOSSARY.md`](../ba/03-DOMAIN-GLOSSARY.md), not column names.

## 3. Errors describe the business problem

Written for Minh (Analyst), who is not a developer.

| Bad | Good |
|---|---|
| `IntegrityError: duplicate key value violates unique constraint` | Row 47: this order line was already imported in batch #12 on 3 March. |
| `422 Unprocessable Entity` | Row 91, column "Ship Fee": expected a number, found "N/A". |
| `Foreign key violation` | Row 12: channel SKU `TX-889` is not mapped to an internal SKU yet. |

Every error names **where** (row and column), **what** is wrong, and
**what to do next**. Never a stack trace, a constraint name, or a raw
status code. Error text never contains a personal-data value
([`../security/DATA-PRIVACY.md`](../security/DATA-PRIVACY.md)).

## 4. Four states, always

Every screen and data component implements:

| State | Requirement |
|---|---|
| **Loading** | Skeleton matching the eventual layout — reuse the existing `components/Pending/*` pattern. No layout shift. |
| **Empty** | Explain *why* it is empty and give the next action. Never a bare "No data". |
| **Error** | What failed, whether it is retryable, and a retry control. |
| **Happy** | The data. |

This is a Definition of Done item, not a polish item. Component tests for
these states arrive with the Vitest harness
([`../architecture/adr/ADR-0007-FRONTEND-TEST-RUNNER.md`](../architecture/adr/ADR-0007-FRONTEND-TEST-RUNNER.md)).

## 5. Destructive and historical actions are explicit

- Anything that changes reported history (correcting a past cost, editing a
  fee rule, unlocking a period, recalculating) states **what will change**
  before confirmation — not a generic "Are you sure?".
- Irreversible actions require typed confirmation, following the existing
  `UserSettings/DeleteConfirmation` pattern.
- Actions the current role cannot perform are **visible but disabled with a
  reason**, not hidden — hiding teaches users the feature does not exist.
  (Server-side enforcement is the real control regardless;
  [`../security/AUTHORIZATION-POLICY.md`](../security/AUTHORIZATION-POLICY.md) §3.)

## 6. Bulk work is designed for, not retrofitted

Minh may face hundreds of unmapped SKUs or failed rows.

- Every list has filtering, sorting, pagination and a visible total count.
- Multi-select with bulk actions wherever a per-row action exists and
  repeating it 200 times would be absurd.
- Long operations run in the background with visible progress and a
  terminal state. A background job that finishes silently is a bug.
- Exports are available wherever a filtered list exists.

## 7. Suggest, never auto-commit

From the AI/automation boundary in
[`../ba/01-PRODUCT-VISION.md`](../ba/01-PRODUCT-VISION.md):

- The system may suggest a SKU mapping, a fee classification, a match. It
  never confirms one on the user's behalf.
- Suggestions show their confidence and their reasoning.
- Accepting a suggestion is a deliberate act, attributed to the user who
  made it.

## 8. Money is displayed consistently

- Always with its currency; never a bare number.
- Consistent decimal places within a view.
- Negative amounts are unmistakable (sign **and** colour — never colour
  alone, which fails for colour-blind users and in print).
- Rounding happens once, server-side. The frontend formats; it never
  computes money ([`../architecture/adr/ADR-0003-MONEY-AND-CURRENCY.md`](../architecture/adr/ADR-0003-MONEY-AND-CURRENCY.md)).
- Sign conventions are stated on screen where they could be ambiguous —
  particularly reconciliation variance.

## 9. Dates and periods are unambiguous

- Always state which date a figure is keyed on (order date, settlement
  date, payout date). "March profit" is meaningless without it.
- Show the timezone where it could matter.
- Period boundaries are visible in filters, not implied.

## 10. Build on what is already there

The inherited component library is kept
([`../architecture/adr/ADR-0004-TEMPLATE-INHERITANCE.md`](../architecture/adr/ADR-0004-TEMPLATE-INHERITANCE.md)).
Reuse `components/ui/*` (shadcn/Radix), `Common/DataTable`, `Pending/*`
skeletons, `sonner` toasts, and `react-hook-form` + `zod` form patterns.
Do not introduce a second component library, a second form library, or a
second table implementation.

Charts have no library yet. When one is chosen it is a small ADR, and the
visual system must satisfy §8 (money formatting), colour-blind safety, and
light/dark themes — the app already ships `next-themes`.

## 11. Accessibility floor

Not a stretch goal: keyboard navigation for every interactive element,
visible focus, form labels tied to inputs, sufficient contrast in both
themes, and no meaning conveyed by colour alone. The Radix primitives in
use provide most of this — the obligation is not to break it.

## 12. Performance expectations

- A dashboard view responds in about a second on realistic data; beyond
  that, show progress rather than a frozen screen.
- Long imports and reconciliation runs are background work with status,
  never a blocked request.
- Tables paginate server-side. Never fetch "all orders" to the browser.
