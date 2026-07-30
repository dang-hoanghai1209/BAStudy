# VS-002 — Product Catalog and Internal SKU

**Slice ID:** VS-002 · **Depends on:** VS-001 · **Status:** Not started · **Level:** Specified

## Business objective

Give the seller one canonical product identity that is independent of any
marketplace, so profit can be aggregated per real product rather than per
channel listing.

## BA learning outcome

Distinguishing **identity** from **representation**: the thing you sell
versus how a particular marketplace refers to it. Writing uniqueness,
normalisation and lifecycle rules for master data.

## Business rules required

| Rule | Status |
|---|---|
| Internal SKU codes are unique **per organization**, not globally | Decided (follows ADR-0002) |
| SKU code normalisation: case, whitespace, leading zeros | **Open** — must be answered before coding |
| Can a product exist without a variant? | **Open** |
| Can an internal SKU code be changed after orders reference it? | **Open** — recommend no; supersede instead |
| Deactivate rather than delete | Decided — [historical-data policy](../../architecture/HISTORICAL-DATA-POLICY.md) §6 |
| Are bundles/kits in scope? | **Open (D-11)** — recommend deferring; changes mapping cardinality to 1:N |

## Entities involved

`product`, `product_variant`, `internal_sku`. All carry `organization_id`;
none carry `shop_id` — catalog is organization-wide master data shared
across shops, which is the point of an internal SKU.

`unit_of_measure` only if bundles are brought into scope.

## Expected API capabilities

CRUD for product, variant and internal SKU, organization-scoped. Search by
code or name. Deactivate (not delete). Missing-cost and mapping-status
fields on the SKU read model, populated in VS-003 and VS-006.

Bulk creation via CSV is **not** here — it reuses the import machinery in
VS-004.

## Expected frontend screens

| Screen | Route |
|---|---|
| Products list (searchable `DataTable`) | `/catalog/products` |
| Product detail with variants | `/catalog/products/$productId` |
| Internal SKU detail | `/catalog/skus/$skuId` |
| Create/edit dialogs, deactivate confirmation | in-page |

`/catalog/skus/$skuId` becomes the hub that VS-003 (mappings), VS-006 (cost
timeline) and VS-015 (SKU profitability) all extend. Design it as an
extensible detail page, not a one-off.

## Acceptance criteria

1. Two organizations can hold the same SKU code without collision.
2. A duplicate SKU code within one organization is rejected with a
   field-level error naming the conflict.
3. Deactivating a SKU keeps it visible on historical records but excludes
   it from new mapping choices.
4. Code normalisation is applied consistently on create, update and search
   — searching `sku-001` finds `SKU-001` if the rule says it should.
5. Analyst can create and edit; Viewer cannot.
6. **The `Item` entity is retired in this slice**, in its own dedicated
   commit, and coverage still passes the 90% gate afterwards.

## Test layers

- **Unit** — code normalisation; uniqueness scoping; deactivation semantics.
- **Integration** — CRUD; duplicate rejection; cross-organization 404;
  role matrix.
- **Contract** — client regenerated.
- **Frontend** — list/detail loading, empty, error states.
- **E2E** — create a product with a variant and a SKU; see it in the list.

## Dependencies

VS-001 (organization scoping, roles, audit emit helper).

## Risks

| Risk | Mitigation |
|---|---|
| Over-modelling variants before real data justifies it | Keep variant optional until a real catalog demands it |
| Normalisation rules invented while coding (R-01) | Answer the open question first; it changes stored data |
| Retiring `Item` drops coverage below 90% | Retire only after catalog tests are in place, as a separate commit that CI validates on its own |

## CV evidence

A master-data model with documented uniqueness, normalisation and lifecycle
rules — and the discipline of retiring inherited demo code on a defined
trigger rather than leaving it or deleting it carelessly.

## Explicit exclusions

No inventory quantities. No bundles or kits. No suppliers. No images or
media. No pricing. No channel listings or SKU mapping (VS-003). No cost
data (VS-006). No bulk import (VS-004).
