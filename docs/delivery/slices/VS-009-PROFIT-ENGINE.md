# VS-009 — Profit Calculation Engine

**Slice ID:** VS-009 · **Depends on:** VS-006, VS-007, VS-008 · **Status:** Not started · **Level:** Seeded

> **Seeded spec.** The highest-value and highest-risk slice in the roadmap.
> Business analysis must be complete before Ready — this is the slice where
> guessing a rule produces confident wrong numbers.

## Business objective

Produce a per-order and per-line profit figure that a seller can trust and
trace back to its inputs.

## BA learning outcome

Converting a formula specification into a **versioned, explainable**
calculation; and being able to isolate a disagreement to one of: wrong
requirement, wrong formula, wrong data, wrong code.

## Business rules required — **pending**

| Question | Status |
|---|---|
| The finalized formula | **Open** — [`07-PROFIT-CALCULATION-SPEC.md`](../../ba/07-PROFIT-CALCULATION-SPEC.md) is explicitly a hypothesis |
| **Q7 / Q8** — cost basis and recalculation | **Open — blockers** |
| **Q11** — indirect expense allocation basis, if in scope | **Open** |
| **D-12** — recalculation trigger and who may trigger it | **Open — blocker** |
| Behaviour when an input is missing: fail, park, or partial | **Open** |
| Calculation versioning scheme | **Open** |
| The decision table (cancelled × partial refund × missing COGS) | **Not yet created** — required deliverable of this slice |

## Entities involved

`profit_calculation` (ledger row per order line), `calculation_version`,
`calculation_input_trace` (which cost record, which fee rule versions,
which source rows). The ledger is **append-only** — a recalculation adds an
entry, it never mutates one
([historical-data policy](../../architecture/HISTORICAL-DATA-POLICY.md) §4).

## Expected API capabilities

Trigger calculation for a batch or date range. Get profit for an order or
line **with its full input breakdown** ("explain"). Recalculate with a
stated reason. List calculation versions.

## Expected frontend screens

Profit breakdown panel on order detail; a shared "explain this number"
drawer reused by the dashboard; a permissioned recalculate action showing
what will change before confirming.

## Acceptance criteria

1. The worked example in the profit calculation spec reproduces exactly.
2. Every profit figure exposes the cost record ID and effective date, the
   fee rule versions, and the source row IDs used.
3. A recalculation creates a new ledger entry; the prior entry remains
   readable.
4. A missing COGS produces a flagged, non-silent result.
5. All money arithmetic uses `Decimal`; no `float` appears in any
   calculation path.
6. Recalculating a locked period is refused.
7. A recalculation that changes a reported figure is audited with actor,
   scope and reason.

## Test layers

Unit (each formula term; missing-input paths; decimal precision; the full
decision table); integration (calculate → explain → recalculate; permission
matrix; cross-org 404); contract; frontend (breakdown panel states); E2E
(import → calculate → view breakdown).

**UAT is mandatory and is the point of this slice.** A unit test can be
green while checking the wrong formula — cross-check against hand-worked
examples, per [`../TEST-STRATEGY.md`](../TEST-STRATEGY.md) §"Out of scope
for automated testing".

## Dependencies

VS-006 (cost), VS-007 (orders), VS-008 (normalized charges).

## Risks

| Risk | Mitigation |
|---|---|
| Blocked outright until Q1–Q3, Q7, Q8, D-12 are answered | All are Definition-of-Ready gates |
| Recalculation silently rewrites reported history | Append-only ledger; period locking; audit with reason |
| A green test suite validating the wrong formula | Mandatory UAT against hand-worked examples |
| Explanation payload designed after the fact and unable to reconstruct inputs | Input traces are in the schema from the first migration |

## CV evidence

An explainable calculation engine with a traceability matrix and a decision
table for rule combinations — the artifact
[`07-PROFIT-CALCULATION-SPEC.md`](../../ba/07-PROFIT-CALCULATION-SPEC.md)
currently says does not yet exist.

## Explicit exclusions

No dashboard aggregation (VS-014). No refund handling (VS-010). No
forecasting or simulation. No AI explanation layer. No reconciliation
(VS-012).
