# VS-008 — Fee, Discount, Shipping and Tax Normalization

**Slice ID:** VS-008 · **Depends on:** VS-007 · **Status:** Not started · **Level:** Seeded

> **Seeded spec.** Business analysis must be completed before Ready.
> This slice has the highest business-rule density in the roadmap.

## Business objective

Turn every marketplace's differently-named money line into a normalized,
typed set of amounts that a formula can consume — and keep a record of what
the marketplace originally called it.

## BA learning outcome

Building a classification taxonomy and a versioned rule engine; separating
"what the marketplace called it" from "what it is".

## Business rules required — **pending**

| Question | Status |
|---|---|
| **Q1** — is a discount borne by the seller or the marketplace? | **Open — blocker** |
| **Q2** — is fee calculated on gross or net sales? | **Open — blocker** |
| **Q3** — is fee rounded per item or per order? | **Open — blocker** |
| The fee-type taxonomy itself | **Open** |
| Sign conventions per fee type | **Open** |
| Handling of an unrecognised fee label: reject, park, or default | **Open** |
| Is tax a cost, a pass-through, or excluded? | **Open** |
| Fee rules are effective-dated | Decided — [historical-data policy](../../architecture/HISTORICAL-DATA-POLICY.md) |
| **Q15** — who may edit a fee rule | Draft: Owner only — see [authorization policy](../../security/AUTHORIZATION-POLICY.md) |

## Entities involved

`fee_type` (taxonomy), `fee_rule` (effective-dated, versioned),
`order_charge` / `order_line_charge` (normalized amount, type, original
source label, rule version reference), `discount_allocation`.

All money `NUMERIC(18,4)`; rounding half-up applied once at a documented
boundary ([ADR-0003](../../architecture/adr/ADR-0003-MONEY-AND-CURRENCY.md)).

## Expected API capabilities

CRUD fee rules (effective-dated, Owner-only, reason-mandatory on
supersession). Fee-type catalogue. Normalized charges for an order.
Unrecognised-fee-label report. Re-run normalization for a batch.

## Expected frontend screens

Fee rules admin with version history (`/settings/fee-rules`);
unrecognised-label queue (`/imports/unrecognised-fees`); charges breakdown
on order detail showing source label → normalized type → rule applied.

## Acceptance criteria

1. Every normalized charge names the rule **and rule version** that
   produced it.
2. An unrecognised fee label is surfaced, never silently dropped or
   bucketed as zero.
3. Editing a fee rule creates a new version; existing charges keep pointing
   at the old version.
4. Rounding matches the recorded answer to Q3, proven by a test at a
   half-unit boundary.
5. Seller-funded and marketplace-funded discounts are distinguishable, per
   Q1.
6. Only an Owner can create or supersede a fee rule; the action is audited
   with a reason.

## Test layers

Unit (per fee type; sign conventions; rounding at half-cent boundaries;
seller vs. marketplace discount split; version resolution by date);
integration (rule CRUD, permission checks, re-normalization, cross-org
404); contract; frontend (rules admin, version history, unrecognised
queue); E2E (view an order's charge breakdown).

## Dependencies

VS-007.

## Risks

| Risk | Mitigation |
|---|---|
| Very likely to stall on unanswered Q1–Q3 | Treat all three as hard Definition-of-Ready gates |
| Money rounding done in floats corrupts everything downstream | ADR-0003; explicit boundary tests |
| Fee taxonomy invented from imagined data rather than real statements (B-5) | Obtain real anonymised exports first |
| Rule versioning added later rather than designed in | Effective dating is in the schema from the first migration |

## CV evidence

A fee taxonomy and decision table; a versioned rule engine whose output
explains which rule version produced it. This is the strongest pure-BA
artifact in the roadmap.

## Explicit exclusions

No tax filing or compliance logic. No advertising-cost allocation. No
automatic fetching of marketplace rate cards. No profit calculation
(VS-009).
