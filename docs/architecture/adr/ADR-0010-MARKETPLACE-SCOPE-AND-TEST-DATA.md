# ADR-0010 — MVP marketplace scope, and synthetic test data until real samples exist

**Status:** Accepted
**Date:** 2026-07-26
**Deciders:** Project owner (human)

## Context

Two questions had to be settled before VS-004 (import), VS-008 (fee
normalization) and VS-011 (settlement import) could be specified, because
all three are shaped by the actual file formats they consume:

1. Which marketplaces are in MVP scope?
2. Can real, anonymised sample exports be obtained?

The VS-000 audit raised the second as blocker **B-5**. The Olist Brazilian
E-commerce Dataset — the fallback the pre-audit draft suggested — has
orders and payments but **no COGS, no fee ledger and no settlement
statements**, so it cannot exercise the slices that matter most.

## Decision

### Marketplace scope

**Shopee and TikTok Shop are in MVP scope. Lazada is deferred.**

Two marketplaces, not one: the product's central claim is that a SKU sold
across channels rolls up to one internal SKU and one profit figure. A
single-marketplace MVP cannot demonstrate that, and the mapping and
normalization slices would be built against a degenerate case. Two is the
minimum that proves the concept; a third adds format-handling work without
adding a new capability.

### Test data

**No real marketplace samples are available.** Development proceeds on
**synthetic fixtures modelled on each marketplace's publicly documented
export schema**, subject to these binding conditions:

1. Every fixture is **clearly labelled synthetic**, in the file itself and
   in any documentation or demo that uses it. Synthetic cost, fee or
   settlement figures are never presented as real or reconciled.
2. Fixtures model the **schema** — column names, ordering, encoding,
   delimiter, date and money formats — not merely plausible content. The
   schema is what the code is built against, so it is what must be right.
3. **B-5 remains a live risk** in VS-004, VS-008 and VS-011 until real
   samples arrive. It is downgraded from a blocker to a tracked risk, not
   closed.
4. Where a fixture encodes an assumption about a marketplace's behaviour
   that could not be verified (how a fee line is labelled, how a refund is
   represented, the grain of a settlement statement), that assumption is
   **recorded in the slice spec as an assumption**, not written as fact.
5. When a real sample is eventually obtained, the first action is to
   **diff it against the synthetic fixture** and record what differed. That
   diff is the evidence of what the assumption cost.

### Consequence accepted explicitly

Building against synthetic fixtures means the import, fee and settlement
slices are validated against **our model of the marketplaces**, not against
the marketplaces. Some rework when real data arrives is expected and
budgeted, not a surprise. VS-011 (settlement import) carries the most
exposure, because statement formats vary more than order exports and are
the least publicly documented.

## Alternatives considered

- **Block VS-004 until real samples exist** — rejected. It stalls the
  entire roadmap on an external dependency with no committed date, and the
  import machinery's structure (source/canonical/business layering,
  idempotency, column profiles, validation) is largely format-independent
  and can be built and proven now.
- **One marketplace only** — rejected. See above; it cannot demonstrate
  cross-channel SKU rollup, which is the product's core claim.
- **Three or more marketplaces** — rejected. Additional format handling
  without an additional capability, at a stage where formats are guessed
  anyway.
- **Use the Olist dataset as the primary fixture** — rejected as a
  substitute, though it remains usable for order/order-item structural
  volume testing. It has no fee ledger, no COGS and no settlements, so it
  cannot exercise VS-008 or VS-011 at all.

## Consequences

- Positive: the roadmap is unblocked; import structure can be built and
  tested now.
- Positive: the column-profile design in VS-004 exists precisely to absorb
  format variation, so a schema surprise is a data-entry change rather
  than a code change.
- Trade-off: rework risk concentrated in VS-008 and VS-011, acknowledged
  and tracked.
- Obligation: fixture labelling and assumption-recording are acceptance
  criteria in the affected slices, not good intentions.
- Obligation: revisit this ADR when the first real sample arrives.

## Related

[`../../delivery/RISK-REGISTER.md`](../../delivery/RISK-REGISTER.md) B-5,
[`../../ba/05-MVP-SCOPE.md`](../../ba/05-MVP-SCOPE.md),
[`../../delivery/slices/VS-004-CSV-IMPORT-CENTER.md`](../../delivery/slices/VS-004-CSV-IMPORT-CENTER.md),
[`../../delivery/slices/VS-008-FEE-NORMALIZATION.md`](../../delivery/slices/VS-008-FEE-NORMALIZATION.md),
[`../../delivery/slices/VS-011-SETTLEMENT-IMPORT.md`](../../delivery/slices/VS-011-SETTLEMENT-IMPORT.md),
decision D-07 in [`../../ba/08-BUSINESS-RULES.md`](../../ba/08-BUSINESS-RULES.md).
