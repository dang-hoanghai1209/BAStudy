# VS-012 — Settlement Reconciliation and Matching

**Slice ID:** VS-012 · **Depends on:** VS-009, VS-011 · **Status:** Not started · **Level:** Seeded

> **Seeded spec.** The hardest slice in the roadmap. Business analysis must
> be complete before Ready.

## Business objective

Compare expected payout (what SellerMargin calculated) against actual
payout (what the marketplace paid), and **explain every difference**.

## BA learning outcome

Designing a matching algorithm with tolerance, and classifying variance by
**cause** rather than merely reporting a delta. A number that differs is
only useful if you know why.

## Business rules required — **pending**

| Question | Status |
|---|---|
| Matching key hierarchy (order ID → order + item → amount + date) | **Open** |
| Tolerance threshold for "matched" | **Open** |
| Variance classification taxonomy | **Open** |
| **Q9** — can a reconciled period be locked? | **Open — blocker** |
| **Q14** — how are late-arriving transactions handled? | **Open — blocker** |
| **Q15** — who may confirm a reconciliation? | Draft: Owner only |
| **Q13** — one order spanning multiple payouts | **Open** |

Q9 and Q14 interact with refunds (VS-010) and recalculation (VS-009). They
must be decided together, before coding, not discovered during it.

## Entities involved

`reconciliation_run`, `reconciliation_match`, `reconciliation_variance`,
`period_lock`. All organization- and shop-scoped.

## Expected API capabilities

Run reconciliation for a shop and period. Match and unmatch manually. List
variances with classification. Confirm or close a reconciliation. Lock and
unlock a period. Reconciliation summary.

## Expected frontend screens

Reconciliation workspace with matched / unmatched / variance panes
(`/reconciliation`); variance detail with drill-through to **both** the
order and the statement line; a confirm action gated to Owner; period lock
management (`/settings/periods`).

## Acceptance criteria

1. Every statement line ends in exactly one state: matched, unmatched, or
   classified variance. Nothing disappears.
2. Variance sign convention matches
   [`06-KPI-DICTIONARY.md`](../../ba/06-KPI-DICTIONARY.md) — positive means
   the seller received **more** than expected — and is stated on screen.
3. A confirmed reconciliation is immutable; reopening it is audited with
   actor and reason.
4. A one-order-to-many-payouts case reconciles correctly per Q13.
5. Locking a period prevents recalculation of that period.
6. Only an Owner may confirm, lock or unlock; each is audited with a reason.

## Test layers

Unit (match key precedence; tolerance at the exact boundary; variance
classification; multi-payout allocation); integration (full run over a
fixture order/statement pair; manual match; confirm; lock; recalculation
refused on a locked period; role matrix); contract; frontend (workspace
states, large variance lists); E2E (run reconciliation → resolve one
variance → confirm).

**UAT mandatory.**

## Dependencies

VS-009 (expected payout), VS-011 (actual payout).

## Risks

| Risk | Mitigation |
|---|---|
| Hardest slice in the roadmap; rated "Hard" and correctly so | Do not start until Q9, Q13, Q14, Q15 are all answered |
| Period locking interacts with refunds and recalculation in ways discovered too late | Decide the three together as one policy |
| Statement lines quietly matching nothing and vanishing from view | Acceptance criterion 1; every line has exactly one terminal state |
| Tolerance chosen arbitrarily, hiding real errors | Tolerance is a documented business rule with a rationale |

## CV evidence

"Expected payout vs. actual payout is compared and variance explained" —
the depth criterion sellers actually pay for, and the one that most
distinguishes this project. A reconciliation SOP document.

## Explicit exclusions

No automatic dispute filing with marketplaces. No accounting-system
posting. No double-entry ledger. No cash-flow forecasting. No exception
workflow (VS-013) — this slice raises variances, VS-013 manages them.
