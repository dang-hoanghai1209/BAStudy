# UAT Strategy

**Status:** Draft skeleton (VS-000).

## Purpose

Automated tests prove the code does what the code was told to do. UAT
proves the code does what the **business** needed — these are not the
same claim, and a slice is not "done" on green tests alone (see
[`../../CLAUDE.md`](../../CLAUDE.md) §9 and
[`../delivery/DEFINITION-OF-READY-DONE.md`](../delivery/DEFINITION-OF-READY-DONE.md)).

A reusable prompt for producing a UAT note is in
[`../prompts/06-UAT-NOTE.md`](../prompts/06-UAT-NOTE.md).

## UAT approach per slice

For each vertical slice, produce a short UAT note containing:
1. The user story / acceptance criteria being verified
2. The scenario walked through (steps taken, as a real user would take
   them)
3. Actual result vs. expected result
4. Pass/fail, and if fail — which of requirement, formula, data, or code
   was at fault (see the diagnostic split in
   `07-PROFIT-CALCULATION-SPEC.md`)

This does not need to be a heavyweight document — a few lines per
acceptance criterion is enough, but it must exist and must be honest
about failures, not just successes.

## Test data

Where a realistic public dataset helps (e.g. for import/order-shaped
data), it may be used for structural testing, but any dataset lacking
real COGS/fee data must have that data clearly marked as **synthetic**,
never presented as production or reconciled figures.

## Six-criteria "is this actually deep" checklist

A version of SellerMargin worth calling done for a phase should be able
to demonstrate:

- Importing the same file twice does not create duplicate data
- A multi-channel SKU maps correctly to one internal SKU
- A cost change over time does not overwrite history
- A partial refund still calculates every component correctly
- A user can trace a profit figure back to its source row
- Expected payout vs. actual payout is compared and variance explained
- A business rule change has an audit trail
- A user from Shop A cannot read Shop B's data
- UAT, a decision table, and a traceability matrix all exist
- There is a demo of one real business scenario end to end

These are evaluation criteria for the project as a whole, not a single
slice — track slice-level UAT here and roll up against this list
periodically.
