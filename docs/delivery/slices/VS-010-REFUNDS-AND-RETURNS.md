# VS-010 — Refunds, Returns and Cancellations

**Slice ID:** VS-010 · **Depends on:** VS-009 · **Status:** Not started · **Level:** Seeded

> **Seeded spec.** Business analysis must be completed before Ready.

## Business objective

Make profit survive money going back out — at the right time, with the
right fee treatment, and without corrupting the original sale's record.

## BA learning outcome

Modelling reversals and their **timing**; recognising that "revenue minus
refund" is a business-policy question, not arithmetic.

## Business rules required — **pending**

| Question | Status |
|---|---|
| **Q4** — when is a refund recognized: order date, refund date, or statement date? | **Open — blocker** |
| **Q5** — does a refund return the marketplace's commission? | **Open — blocker** |
| **Q6** — is returned merchandise restocked? | **Open** — likely N/A, no inventory scope |
| Partial refund allocation across lines | **Open** |
| Cancellation before shipment vs. return after delivery | **Open** |
| Refund exceeding the original line amount | **Open** |
| **Q14** — late-arriving refunds against a closed period | **Open** |

## Entities involved

`refund`, `refund_line`, `adjustment`, `cancellation`, plus extension of
`order_status_history`. Refunds never mutate the original order line; they
are separate records that the calculation consumes.

## Expected API capabilities

Import refunds through the VS-004 pipeline with a refund file type. List
refunds for an order. Recalculated profit reflecting refunds. Refund-impact
report per period.

## Expected frontend screens

Refunds tab on order detail; refunds list (`/refunds`); profit breakdown
extended to show the refund term **separately** from the original sale, so
the original figure stays visible.

## Acceptance criteria

1. A partial refund recalculates **every** component correctly, not just
   revenue.
2. Refund recognition date matches the recorded answer to Q4 and drives the
   period the impact lands in.
3. Commission reversal follows the recorded answer to Q5.
4. A full refund yields the arithmetically expected net profit for that
   order.
5. The original sale record is unchanged; the refund is additive.
6. A refund arriving for a locked period follows the recorded Q14 rule.

## Test layers

Unit (partial, full and over-refund; commission reversal on and off; period
assignment; allocation across lines); integration (import refund file →
recalculation; locked-period behaviour); contract; frontend (refunds tab
states); E2E (refund import → order shows adjusted profit).

## Dependencies

VS-009.

## Risks

| Risk | Mitigation |
|---|---|
| Late-arriving refunds reopen already-reported periods | Q14 and period locking (Q9) answered together before coding |
| Partial refunds handled for revenue but not for fees or COGS | Explicit per-component tests |
| Refund modelled as a mutation of the order | Schema forbids it; refunds are separate records |

## CV evidence

"A partial refund still calculates every component correctly" — one of the
six depth criteria, and the case most portfolio projects skip. Documented
timing policy with its consequences.

## Explicit exclusions

No inventory restocking. No RMA or returns-logistics workflow. No customer
communication. No automatic dispute handling. No reconciliation (VS-012).
