# AS-IS / TO-BE Process

**Status:** Draft skeleton (VS-000), carried over from prior research.
Needs validation against real seller workflows.

## AS-IS (current manual process, hypothesized)

```
Download report from marketplace
  → Manual Excel processing
    → Merge in cost of goods
      → Calculate fees
        → Calculate profit
          → Produce a report
```

Known weaknesses of this process:
- Manual, repeated per marketplace, per period
- No versioning of cost data — if a cost changes, history is silently
  affected or ignored inconsistently
- No systematic validation of the source export before it's used
- No reconciliation step against actual payout — profit is assumed, not
  confirmed against money received

## TO-BE (target process)

```
Upload CSV
  → Validate data
    → Map SKUs
      → Apply costs and fees
        → Calculate profit
          → Dashboard
```

With reconciliation added once the profit engine is trustworthy:

```
... → Dashboard
        → Import settlement/payout data
          → Auto-match against expected payout
            → Surface and classify variance
```

## Gap analysis (high level)

| Gap in AS-IS | TO-BE capability that closes it |
|---|---|
| No canonical SKU across channels | Internal SKU catalog (VS-002) + Channel Mapping (VS-003) |
| No validation before use | Import Center (VS-004) + validation and error handling (VS-005) |
| Cost changes silently affect history | Time-versioned Cost History (VS-006) |
| Fee lines differ per marketplace and are not comparable | Fee normalization and versioned rules (VS-008) |
| No traceability | Every calculated figure links to its source rows and rule versions (VS-009, exposed in VS-015) |
| No reconciliation | Settlement import (VS-011) + reconciliation and variance (VS-012) |
| Problems found but never resolved | Exception management (VS-013) |

Slice numbers follow
[`../delivery/VERTICAL-SLICE-ROADMAP.md`](../delivery/VERTICAL-SLICE-ROADMAP.md),
which was renumbered at VS-000 when the audit split import and mapping into
separate slices.

## Automated steps (target)

SKU mapping suggestion, data validation, fee application, profit
calculation, and payout matching are all candidates for automation.
None of them should auto-confirm without a human check where the action
is irreversible or affects trust in historical data (see AI boundaries
in `01-PRODUCT-VISION.md`).
