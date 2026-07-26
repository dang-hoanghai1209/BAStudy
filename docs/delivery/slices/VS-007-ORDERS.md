# VS-007 — Orders and Order Lines (canonical)

**Slice ID:** VS-007 · **Depends on:** VS-005 · **Status:** Not started · **Level:** Seeded

> **Seeded spec.** Objective, entities, acceptance criteria, dependencies,
> risks and exclusions are carried from the roadmap. The business-analysis
> section is deliberately incomplete — it must be completed, with the
> owner, before this slice can pass the Definition of Ready.

## Business objective

Turn validated import rows into canonical orders and order lines that every
downstream calculation reads from, regardless of which marketplace the data
came from.

## BA learning outcome

Designing a canonical model across heterogeneous source formats; defining
natural keys and an order lifecycle that survives re-import.

## Business rules required — **pending**

| Question | Status |
|---|---|
| Natural key for an order and an order line, per marketplace | **Open (D-09)** |
| Re-import containing a changed version of an existing order: update in place, or new version? | **Open (D-09)** |
| Order status vocabulary, and which statuses count as revenue | **Open** |
| Order date vs. payment date vs. ship date — which drives period assignment? | **Open (D-08)** |
| Can one order span multiple shops? | **Open** |
| Timezone and period-boundary policy | **Open (D-08)** |

## Entities involved

`order`, `order_line`, `order_status_history`, plus lineage references back
to `import_batch` and `import_source_row`. All carry `organization_id` and
`shop_id`. All money is `NUMERIC(18,4)` with an explicit currency.

## Expected API capabilities

List orders with filters (shop, date range, status, SKU). Order detail with
lines. Order → source row lineage. Order counts per batch.

## Expected frontend screens

Orders list with filters (`/orders`); order detail with line items and a
source panel (`/orders/$orderId`). The detail page is extended by VS-008
(charges), VS-009 (profit) and VS-010 (refunds) — design it to grow.

## Acceptance criteria

1. Re-importing an existing order updates or versions it per the recorded
   rule — never a silent duplicate.
2. Every order line links to the raw source row it came from.
3. Orders are visible only within their organization.
4. Status transitions are recorded with a timestamp.
5. Period assignment follows the recorded date rule, provably.

## Test layers

Unit (natural-key derivation, status mapping, period assignment at
boundaries); integration (import→order projection, re-import, filters, role
matrix, cross-org 404); contract; frontend (list/detail states); E2E
(import fixture → orders appear → drill to source row).

## Dependencies

VS-004, VS-005. VS-003 for SKU resolution on lines.

## Risks

| Risk | Mitigation |
|---|---|
| Marketplace quirks force per-marketplace branching into a "canonical" model | Keep marketplace specifics in the import/mapping layer, not in `order` |
| Period assignment decided by code rather than by rule | D-08 is a Definition-of-Ready gate |
| Order volume makes unpaginated queries unusable | Server-side pagination from the first implementation |

## CV evidence

A canonical model across heterogeneous sources with full lineage — the
foundation of the traceability claim, completed in VS-015.

## Explicit exclusions

No fee or discount normalization (VS-008). No profit (VS-009). No refunds
(VS-010). No user editing of orders. No settlement data (VS-011).
