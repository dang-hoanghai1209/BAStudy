# VS-006 — Time-Effective Cost History

**Slice ID:** VS-006 · **Depends on:** VS-002, VS-005a · **Status:** Not started · **Level:** Specified

## Business objective

Record what a SKU cost at each point in time, so that historical profit
stays correct when today's cost changes.

## BA learning outcome

Temporal (effective-dated) data modelling, and being able to articulate why
overwriting a cost destroys business truth rather than merely losing an
audit detail.

## Business rules required

| Rule | Status |
|---|---|
| **Q7** — COGS at time of sale, or current cost? | **Open — hard blocker** |
| **Q8** — does editing an old cost recalculate historical orders? | **Open — hard blocker** |
| Effective ranges for one SKU never overlap | Decided |
| At most one open-ended (current) record per SKU | Decided |
| Ranges are half-open: `effective_from` inclusive, `effective_to` exclusive | Decided |
| An order with no covering cost record raises a reportable exception, never a zero | Decided |
| Correction is append-and-supersede, never in-place update | Decided |
| Does cost include inbound shipping, duties, tax? | **Open** |
| Cost currency | Decided — organization currency; mismatch rejected |

Q7 and Q8 are the two questions this entire slice exists to answer
correctly. Coding before they are answered would mean guessing the
product's central promise.

## Entities involved

`cost_history`: `internal_sku_id`, `cost_amount` `NUMERIC(18,4)`,
`currency`, `effective_from`, `effective_to` (nullable), `created_by`,
`created_at`, `note`, `organization_id`. Optionally `cost_import_batch`
for bulk uploads.

**Never** a `product.cost` column. That is the anti-pattern this slice
exists to prevent.

## Expected API capabilities

List cost history for a SKU. Add a cost effective from a date. Correct a
past record by superseding it (permissioned, reason mandatory). Resolve
"the cost applicable at date X". Bulk cost upload (reusing VS-004
machinery). Missing-cost report.

## Expected frontend screens

| Screen | Route |
|---|---|
| Cost timeline | on `/catalog/skus/$skuId` |
| Add-cost dialog with effective date | in-page |
| Missing-cost report | `/costs/missing` |
| Bulk cost upload | reuses the import wizard |

The UI must read as a **timeline from the first glance**. If it looks like
a single editable "cost" field, the model's intent has been lost in the
interface.

## Acceptance criteria

1. Adding a new cost never mutates an existing row — proven by asserting
   the prior row is byte-identical afterwards.
2. Effective ranges for one SKU never overlap; an attempted overlap is
   rejected.
3. Resolving cost for a date returns the record **covering that date**, not
   the most recent one — verified at exact boundary instants.
4. A SKU with no cost covering an order date appears in the missing-cost
   report and does not silently produce zero.
5. Correcting a past cost behaves exactly as the recorded answer to Q8
   says, and there is a test proving that behaviour either way.
6. Correcting a past cost emits `cost.superseded` with a mandatory reason.
7. Only an Owner may correct a past record; an Analyst may add a new one.

## Test layers

- **Unit** — range resolution including both boundaries; overlap rejection;
  the no-covering-record case; supersession preserving the original.
- **Integration** — add, correct, list, bulk upload; role matrix;
  cross-organization 404; audit emission.
- **Contract** — client regenerated.
- **Frontend** — timeline with one record, many records, none; add-cost
  validation.
- **E2E** — add a cost, see it on the timeline, confirm history unchanged.

## Dependencies

VS-002 (internal SKUs). **VS-005a** — the backend restructure lands before
this slice, so costs are the first domain built in the new structure rather
than the last added to the flat one.

## Risks

| Risk | Mitigation |
|---|---|
| **R-03** — a cost edit silently rewriting history; the single most damaging failure mode for this product's credibility | Append-only supersession; dedicated tests; permissioned correction; audit with reason |
| Timezone / date-boundary ambiguity on `effective_from` (D-08) | Answer D-08 before coding; test at exact boundaries |
| UI implies a single mutable cost field | Timeline-first design; reviewed as an acceptance criterion |

## CV evidence

An effective-dated design with a test that proves history is preserved —
"a cost change over time does not overwrite history" is one of the
project's six depth criteria. An ADR-backed append-only correction pattern.

## Explicit exclusions

No FIFO or weighted-average inventory costing (explicitly Won't-have in
[`../../ba/05-MVP-SCOPE.md`](../../ba/05-MVP-SCOPE.md)). No supplier price
lists. No landed-cost allocation. No currency conversion. No profit
calculation (VS-009) — this slice supplies an input, it does not consume it.
