# ADR-0002 — Tenancy Model: organization_id everywhere, shop_id where shop-specific

**Status:** Accepted
**Date:** 2026-07-26
**Deciders:** Project owner (human), recorded at VS-000 approval

## Context

SellerMargin is multi-tenant: several sellers' data lives in one database.
A seller may operate more than one shop, and a shop belongs to exactly one
marketplace presence. The audited repository has **no tenancy at all** —
[`../CURRENT-STATE.md`](../CURRENT-STATE.md) confirms only `user` and `item`
tables exist, with resource access done by comparing `item.owner_id` inline
in each route handler.

Tenancy shape has to be settled before VS-001, because it determines the
primary key and foreign key on every table added afterwards. Changing it
later means re-migrating every business table.

Three options were considered.

## Decision

**Every business table carries `organization_id`. Any table describing
something shop-specific additionally carries `shop_id`.**

- `organization` is the tenancy boundary. Users reach data only via a
  `membership` (user × organization × role).
- `shop` belongs to exactly one organization.
- `shop_id` is added where the record is genuinely per-shop (orders,
  imports, statements, reconciliation runs). It is not added to
  organization-wide master data (products, internal SKUs, cost history,
  fee rules) unless a business rule later requires per-shop variation.
- Both columns are non-nullable where they apply, and indexed.
- Enforcement is a **shared scoping dependency** in `app/api/deps.py`, not
  a per-handler check. A handler that forgets the dependency should fail
  review, and tenant-isolation tests exist to catch it.
- A request for a record outside the caller's organization returns **404,
  not 403** — a 403 confirms the record exists.

## Alternatives considered

- **Shop as the sole tenancy boundary** — rejected. Products, internal
  SKUs and costs are deliberately shared across a seller's shops; that is
  the entire point of an internal SKU. Shop-only tenancy would force
  duplication of master data per shop and make cross-channel profit
  aggregation — the product's core value — awkward.
- **Schema-per-tenant or database-per-tenant** — rejected. Strong
  isolation, but migration and query complexity out of proportion to a
  single-developer project, and it complicates the cross-tenant-free
  reporting this system does not need.
- **PostgreSQL row-level security as the primary mechanism** — rejected as
  *primary*. RLS is valuable defence in depth and may be added later, but
  making it the only enforcement puts business-critical access control in
  a place that is easy to bypass in tests and hard to reason about in code
  review. Application-level checks stay authoritative.

## Consequences

- Positive: one predictable rule for every new table; a single dependency
  to test exhaustively; master data shared across shops for free.
- Positive: tenant-isolation tests become mechanical — for every endpoint,
  a foreign-organization ID must yield 404.
- Trade-off: two scope columns on some tables is mild redundancy
  (`shop_id` implies `organization_id`). Accepted deliberately: it keeps
  every query filterable by organization without a join, and lets the
  scoping dependency apply one uniform filter.
- Trade-off: the denormalized `organization_id` must be kept consistent
  with the shop's organization. Enforced by a composite foreign key or a
  check at write time, and covered by a test.
- Obligation: VS-001 must ship the scoping dependency and the isolation
  test suite together — not the entities alone.

## Related

[`ADR-0006-AUTHORIZATION-MODEL.md`](ADR-0006-AUTHORIZATION-MODEL.md),
[`../TARGET-ARCHITECTURE.md`](../TARGET-ARCHITECTURE.md) §4,
[`../../security/AUTHORIZATION-POLICY.md`](../../security/AUTHORIZATION-POLICY.md),
[`../../delivery/slices/VS-001-ORGANIZATION-AND-SHOP.md`](../../delivery/slices/VS-001-ORGANIZATION-AND-SHOP.md).
