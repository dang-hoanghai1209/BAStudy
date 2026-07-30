# ADR-0003 — Money representation and single-currency MVP

**Status:** Accepted
**Date:** 2026-07-26
**Deciders:** Project owner (human), recorded at VS-000 approval

## Context

SellerMargin exists to produce trustworthy money figures. The audited
repository contains **no money columns at all** — the only tables are
`user` and `item`, so every monetary decision is still open and free to
make correctly.

Two decisions had to be settled before any slice writes an amount:

1. How money is represented in storage, in Python, and over the wire.
2. Whether multiple currencies are supported from the start.

Both are expensive to change later: the first means re-migrating every
amount column, the second means re-deriving every calculated figure.

## Decision

### Representation

- **PostgreSQL:** `NUMERIC(18, 4)` for every monetary column. Four decimal
  places, so per-unit values and fee rates that carry sub-minor-unit
  precision survive intermediate arithmetic.
- **Python:** `decimal.Decimal` end to end. **`float` is forbidden for
  money at every layer** — models, services, aggregation, serialization and
  test fixtures.
- **API:** money is serialized as a JSON **string**, not a JSON number, to
  avoid IEEE-754 round-tripping in JavaScript. The generated TypeScript
  client therefore types money as `string`; the frontend formats for
  display and never performs money arithmetic.
- **Currency:** every monetary record stores its currency code (ISO 4217)
  explicitly, even though only one is supported at MVP. Storing it now
  costs nothing and removes the migration when multi-currency arrives.

### Rounding

- Rounding is **half-up** (`ROUND_HALF_UP`), to the currency's minor unit.
- Rounding is applied **once, at a documented boundary** — at the point a
  figure becomes a reported result — never repeatedly in intermediate
  steps.
- Where a marketplace's own rounding differs (per item vs. per order —
  mandatory question #3), the marketplace's behaviour is modelled
  explicitly as a fee-rule property rather than assumed to match ours.
- Every rounding boundary is covered by a test at a half-unit value.

### Currency scope

- **Single-currency MVP.** One currency per organization, set at
  organization creation and immutable thereafter without an explicit
  migration procedure.
- A record whose currency does not match the organization's currency is
  **rejected at import with a validation error** — never silently coerced
  or converted.
- **No FX rates, no conversion, no multi-currency aggregation** is
  implemented. This is deferred deliberately and recorded here rather than
  left implicit.

## Alternatives considered

- **Integer minor units (store cents as `BIGINT`)** — a common and
  defensible choice; rejected because fee percentages and per-unit costs
  routinely need sub-minor-unit precision, which forces a second scaling
  convention anyway. `NUMERIC` keeps one representation for all monetary
  values and matches what a reviewer reading the database expects.
- **`float` / `double precision`** — rejected outright. Binary floating
  point cannot represent decimal money exactly; it is the single most
  common source of "the totals are off by a cent" defects, and this project
  is specifically about numbers being trustworthy.
- **Multi-currency from day one** — rejected for the MVP. It multiplies the
  complexity of the profit engine (which rate, at which date, for which
  leg) and of reconciliation (payout currency vs. order currency) before
  the single-currency path is even proven. Deferred, not refused.
- **Leaving currency off the schema entirely** — rejected. Adding a
  currency column to every monetary table later is exactly the migration
  this ADR is trying to avoid.

## Consequences

- Positive: exact decimal arithmetic; no cent-drift class of defect.
- Positive: the multi-currency migration path is a feature addition, not a
  schema rewrite, because currency is already stored.
- Trade-off: money as a JSON string is slightly awkward for frontend
  developers and must be documented in the API contract policy so it is not
  "fixed" by someone later.
- Trade-off: `Decimal` arithmetic is slower than `float`. Irrelevant at
  this scale.
- Obligation: a lint or review rule should catch `float` creeping into
  monetary code paths. Until one exists, it is a review checklist item.
- Obligation: an organization's currency being immutable must be enforced
  and tested in VS-001.

## Related

[`../HISTORICAL-DATA-POLICY.md`](../HISTORICAL-DATA-POLICY.md),
[`../DATA-MODEL.md`](../DATA-MODEL.md),
[`../API-CONTRACT-POLICY.md`](../API-CONTRACT-POLICY.md),
[`../../ba/07-PROFIT-CALCULATION-SPEC.md`](../../ba/07-PROFIT-CALCULATION-SPEC.md),
mandatory questions #2 and #3 in
[`../../ba/08-BUSINESS-RULES.md`](../../ba/08-BUSINESS-RULES.md).
