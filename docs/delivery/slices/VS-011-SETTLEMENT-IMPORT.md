# VS-011 — Settlement / Payout Import

**Slice ID:** VS-011 · **Depends on:** VS-005 · **Status:** Not started · **Level:** Seeded

> **Seeded spec.** Business analysis must be completed before Ready.
> Proceeds on **synthetic statement fixtures** per
> [ADR-0010](../../architecture/adr/ADR-0010-MARKETPLACE-SCOPE-AND-TEST-DATA.md)
> (decided 2026-07-26) — but this slice carries the **highest R-26
> exposure** in the roadmap, because settlement formats are the least
> publicly documented. Every fixture assumption must be recorded as an
> assumption in this spec before coding.

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
| **R-26 (was B-5)** — statement formats vary more than order exports; synthetic fixtures here are the roadmap's largest divergence exposure | ADR-0010 conditions apply strictly: schema-faithful fixtures, every format assumption recorded in this spec, first real statement diffed against the fixture and the delta recorded |
| Statement grain misunderstood (per order? per transaction? per day?) | Record the assumed grain explicitly per marketplace; design `statement_line` so a grain correction is a mapping change, not a schema change |
| Personal data in statements | Same P-1 policy as VS-004: hash/truncate at ingest ([data privacy](../../security/DATA-PRIVACY.md) §0) |

## CV evidence

A second source of truth modelled independently of the first — the
groundwork for reconciliation, which is the capability most portfolio
projects never reach.

## Explicit exclusions

No matching or reconciliation logic (VS-012). No bank-feed integration. No
payment-gateway connection. No multi-currency conversion. No dispute
handling.
