# Profit Calculation Specification (v0 — pre-slice)

**Status:** Skeleton only. This document becomes authoritative only once
VS-004 (Cost History) and the profit-calculation slice have answered the
mandatory business questions below. Until then, treat every formula here
as a hypothesis to validate, not an implemented contract.

## Worked example (illustrative target, not yet implemented)

```
Revenue:            500,000
COGS:               250,000
Marketplace Fee:     40,000
Shipping:            20,000
Advertising:          30,000
------------------------------
Expected Profit:    160,000
```

If an implemented result differs from an expected figure like this, the
cause must be isolated to one of: wrong requirement, wrong formula,
wrong data, or wrong code — not patched over.

## Draft formula

```
Order Profit = Net Revenue
             − COGS (at time of sale)
             − Marketplace Fees
             − Shipping Cost
             − Allocated Advertising Cost
             ± Refund/Adjustment impact
```

Each term above needs a business rule before it is implementable. See
`08-BUSINESS-RULES.md` for the catalogue and the mandatory questions
that must be answered per term (e.g. is COGS the cost at time of sale or
current cost; how is advertising allocated — by day, SKU, or order).

## Decision Table

**Not yet created.** A Decision Table enumerating rule combinations
(e.g. cancelled order × partial refund × missing COGS) must be produced
as part of the slice that implements profit calculation — this file
should link to it once it exists, not attempt to enumerate it here in
advance without the underlying business rules confirmed.

## Traceability requirement

Every number produced by this calculation must be traceable back to:
1. The source import row(s) it came from
2. The cost history record(s) used, and their effective date
3. The fee rule(s) applied, and their effective date
4. The calculation version/formula revision used

This is a hard requirement, not an optimization — see
[`../../CLAUDE.md`](../../CLAUDE.md) §9, the
[historical-data policy](../architecture/HISTORICAL-DATA-POLICY.md) §8, and
the six depth criteria in [`09-UAT-STRATEGY.md`](09-UAT-STRATEGY.md).

## Money representation

All amounts here are `NUMERIC(18,4)` in storage and `decimal.Decimal` in
Python — never `float`, at any layer. Rounding is half-up, applied once at
a documented boundary. Single currency for the MVP, stored explicitly. See
[ADR-0003](../architecture/adr/ADR-0003-MONEY-AND-CURRENCY.md).
