# VS-015 — SKU and Order Drill-Down / Traceability Explorer

**Slice ID:** VS-015 · **Depends on:** VS-009, VS-014 · **Status:** Not started · **Level:** Seeded

> **Seeded spec.** Business analysis must be completed before Ready.

## Business objective

Let a user follow any number on screen down to the source row and the rule
version that produced it — turning traceability from a database property
into a usable feature.

## BA learning outcome

Making traceability **demonstrable**. Claiming a system is traceable is
cheap; showing a reviewer the path from a dashboard figure to a raw CSV row
is the evidence.

## Business rules required — **pending**

| Question | Status |
|---|---|
| What lineage must be exposed, and to which roles | **Open** |
| **P-4** — who may view unmasked personal data in a source row | **Open** — [data privacy](../../security/DATA-PRIVACY.md) |
| Retention window for source rows (affects how far back drill-down works) | **Open (P-3)** |

## Entities involved

**None new.** Read paths across `profit_calculation` →
`cost_history` / `fee_rule` version / `order_line` → `import_source_row` →
`import_batch`.

## Expected API capabilities

Full lineage for a profit figure. SKU profitability detail across orders
and periods. Order 360 view. An "explain this number" payload assembled
from the VS-009 input traces.

## Expected frontend screens

SKU detail extended with profit over time, cost timeline, mappings and
orders (`/catalog/skus/$skuId`); order detail extended with a lineage
panel; a shared **"explain this number" drawer** reusable from the
dashboard — the traceability primitive of the whole product.

## Acceptance criteria

1. From any dashboard figure, the contributing orders are at most two
   clicks away.
2. From an order's profit figure, the exact cost record, fee rule versions
   and source rows used are visible.
3. **Drill-down totals equal the aggregate they came from**, exactly.
4. Lineage respects tenant scope and personal-data masking.
5. A figure whose source row has been purged (once retention exists) says
   so, rather than showing a broken link.

## Test layers

Unit (lineage assembly); **integration — the aggregate/detail invariant:
drill-down sums equal the aggregate, tested with generated data, not one
hand-picked case**; contract; frontend (lineage panel states, masked data);
E2E (dashboard → SKU → order → raw source row).

## Dependencies

VS-009 (input traces), VS-014 (the aggregates to drill from).

## Risks

| Risk | Mitigation |
|---|---|
| Aggregate/detail mismatch — the classic reporting defect | An explicit invariant test over generated data, not spot checks |
| Personal data exposed in the raw-row view | Masked by default; revealing is permissioned and audited |
| Lineage impossible to assemble because VS-009 did not store enough | Input traces designed in VS-009's first migration, not added later |

## CV evidence

"A user can trace a profit figure back to its source row" — the depth
criterion that most distinguishes this project from a CRUD portfolio app,
demonstrable live in a demo rather than asserted in a README.

## Explicit exclusions

No ad-hoc query interface. No export from drill-down (VS-018). No bulk
lineage extraction. No new calculation logic — this slice reads what
VS-009 produced.
