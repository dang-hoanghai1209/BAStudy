# VS-003 — Marketplace SKU Mapping

**Slice ID:** VS-003 · **Depends on:** VS-002 · **Status:** Not started · **Level:** Specified

## Business objective

Link each marketplace's own SKU to one internal SKU, so that sales of the
same product across several channels roll up to one product's profit.

## BA learning outcome

Designing a **human-in-the-loop matching workflow**: what a system may
suggest, what a human must confirm, and what a wrong auto-merge costs.
This is the project's first concrete application of the automation
boundary in [`../../ba/01-PRODUCT-VISION.md`](../../ba/01-PRODUCT-VISION.md).

## Business rules required

| Rule | Status |
|---|---|
| One channel SKU maps to exactly one internal SKU at a time | Decided |
| The system may suggest but never auto-confirm a mapping | Decided — vision doc |
| Confidence threshold below which only a suggestion is offered | **Open** |
| Is remapping retroactive to already-imported orders? | **Open (D-06)** — blocks coding; changes whether historical profit moves |
| Who may confirm a mapping | Decided — Owner and Analyst |
| Mapping changes are audited with a reason | Decided — `mapping.remapped` is reason-mandatory |

D-06 is the blocking one. If remapping is retroactive, mapping must be
effective-dated like cost history; if not, orders keep the mapping they had
at import. That is a schema difference, not a preference.

## Entities involved

`channel_listing` (how a product appears on a channel: channel SKU, shop,
title as seen on the marketplace), `channel_sku_mapping` (channel listing →
internal SKU, with confidence, actor, timestamps, and effective dating if
D-06 says retroactive).

Both carry `organization_id`; `channel_listing` also carries `shop_id`.

## Expected API capabilities

List unmapped channel SKUs (the work queue). Create, update and remove a
mapping. Bulk-confirm suggestions. A suggestions endpoint using
**deterministic string similarity** — no ML. Mapping history for a channel
SKU.

## Expected frontend screens

| Screen | Route |
|---|---|
| Mapping workbench: unmapped queue, suggestion, confirm/reject | `/catalog/mappings` |
| Mapping list with filters | `/catalog/mappings/history` |
| Mapping history drawer | on SKU detail |

The workbench is the first screen where **bulk work is mandatory** — 200
unmapped SKUs cannot be a 200-click task
([`../../product/UX-PRINCIPLES.md`](../../product/UX-PRINCIPLES.md) §6).

## Acceptance criteria

1. A low-confidence suggestion is never applied without an explicit confirm.
2. Every suggestion displays its confidence and why it matched.
3. Remapping records who changed it, when, from what, and why.
4. The unmapped-SKU count on the dashboard health tile drops when a mapping
   is confirmed.
5. The same channel SKU cannot hold two active mappings.
6. Bulk-confirming N suggestions produces N audit events, not one.

## Test layers

- **Unit** — suggestion scoring; threshold boundary (just above and just
  below); conflict detection.
- **Integration** — confirm, reject, remap, bulk-confirm, duplicate-mapping
  rejection, role matrix, cross-organization 404.
- **Contract** — client regenerated.
- **Frontend** — workbench empty state ("nothing to map" is a *good* state
  and must look like one), loading, error.
- **E2E** — map an unmapped channel SKU end to end.

## Dependencies

VS-002 (internal SKUs must exist). Channel SKUs are created manually here;
they arrive in bulk from VS-004, which is why VS-004 depends on this slice
rather than the reverse.

## Risks

| Risk | Mitigation |
|---|---|
| Auto-merge on weak similarity corrupts profit aggregation irreversibly | Confirmation is mandatory; threshold is a documented rule; every mapping is attributable |
| Ordering temptation: mapping feels more useful *after* import exists, inviting a merge with VS-004 | Keep separate. VS-004 depends on this |
| D-06 answered late, forcing a schema change mid-slice | Treat D-06 as a hard gate on Definition of Ready |

## CV evidence

A decision table for match / suggest / reject, and an explicit,
documented automation boundary — the system suggests, the human decides,
and the record shows which. Mapping history as a first-class artifact.

## Explicit exclusions

No machine learning. No cross-organization mapping library. No automatic
remapping of historical orders (that is governed by D-06 and, if adopted,
by the recalculation policy in VS-009). No bulk import of channel SKUs
(VS-004).
