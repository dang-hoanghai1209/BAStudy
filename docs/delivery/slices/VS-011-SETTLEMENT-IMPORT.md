# VS-011 — Settlement / Payout Import

**Slice ID:** VS-011 · **Depends on:** VS-005 · **Status:** Not started · **Level:** Seeded

> **Seeded spec.** Business analysis must be completed before Ready.
> **Blocked in practice by B-5** — real settlement statements are needed.

## Business objective

Bring the marketplace's own statement of what it actually paid into the
system as first-class data, independent of what SellerMargin calculated.

## BA learning outcome

Understanding a settlement document as a **distinct source of truth** with
its own grain, vocabulary and timing — not a confirmation of the order data.

## Business rules required — **pending**

| Question | Status |
|---|---|
| **Q12** — does a payout belong to the transaction date, the statement date, or the bank-receipt date? | **Open — blocker** |
| **Q13** — can one order span multiple payouts? | **Open — blocker** |
| **Q10** — how is an adjustment with no Order ID allocated? | **Open** |
| Statement natural key and idempotency | **Open** |
| Statement line taxonomy | **Open** |
| Payout currency vs. order currency | Decided — single currency; mismatch rejected (ADR-0003) |

## Entities involved

`statement`, `statement_line`, `payout`, `financial_transaction`. All carry
`organization_id` and `shop_id`. All money `NUMERIC(18,4)` with explicit
currency.

## Expected API capabilities

Import a statement file through the VS-004 pipeline as a new file type.
List statements per shop and period. Statement detail with lines.
Unallocated-line report.

## Expected frontend screens

Statements list (`/settlements`); statement detail with lines and totals
(`/settlements/$statementId`); an import entry point in the Import Center.

## Acceptance criteria

1. Re-importing the same statement does not duplicate lines.
2. Statement totals reconcile internally — the sum of lines equals the
   stated payout, and a mismatch is reported rather than absorbed.
3. Lines without an Order ID are retained and reported, never discarded.
4. Statements are shop- and period-scoped.
5. Import emits an audit event.

## Test layers

Unit (line classification, internal totals check, natural key); integration
(import, re-import, malformed statement, role matrix, cross-org 404);
contract; frontend (statement detail states); E2E (import statement → view
lines).

## Dependencies

VS-004, VS-005 (import pipeline), VS-001.

## Risks

| Risk | Mitigation |
|---|---|
| **B-5** — statement formats vary more than order exports and are harder to obtain; without real samples this slice is built against invented shapes | Obtain at least one real anonymised statement per marketplace before starting. This is a hard gate |
| Statement grain misunderstood (per order? per transaction? per day?) | Determine from real samples, not assumption |
| Personal data in statements | Same policy as VS-004 ([data privacy](../../security/DATA-PRIVACY.md)) |

## CV evidence

A second source of truth modelled independently of the first — the
groundwork for reconciliation, which is the capability most portfolio
projects never reach.

## Explicit exclusions

No matching or reconciliation logic (VS-012). No bank-feed integration. No
payment-gateway connection. No multi-currency conversion. No dispute
handling.
