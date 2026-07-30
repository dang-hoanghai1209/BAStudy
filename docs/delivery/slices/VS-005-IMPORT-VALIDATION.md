# VS-005 — Import Validation and Error Handling

**Slice ID:** VS-005 · **Depends on:** VS-004 · **Status:** Not started · **Level:** Specified

## Business objective

Make bad data visible and fixable before it can distort profit — instead of
failing a whole file, or worse, letting bad rows through silently.

## BA learning outcome

Writing a **validation-rule catalogue with severities**, and designing
partial-success semantics that a business user can act on.

## Business rules required

| Rule | Status |
|---|---|
| Per-rule severity: blocking vs. warning | **Open** — needed per rule |
| Partial import allowed, or all-or-nothing | **Open** — recommend partial |
| Does an unmapped SKU block a row or park it? | **Open** |
| Reprocessing after the cause is fixed must not duplicate good rows | Decided |
| Who may override a warning | **Open** |

Seed catalogue already drafted in
[`../../ba/08-BUSINESS-RULES.md`](../../ba/08-BUSINESS-RULES.md): missing
Order ID, invalid date, disallowed negative amount, unsupported currency,
duplicate Order ID + Order Item ID, unmapped SKU, missing COGS,
unrecognised fee, missing required column. Each needs a code and a severity.

Currency note: a row whose currency differs from the organization's
currency is **rejected**, never coerced
([ADR-0003](../../architecture/adr/ADR-0003-MONEY-AND-CURRENCY.md)).

## Entities involved

`import_error` (batch, row number, column, rule code, severity, message,
resolution state), `validation_rule` catalogue (may be static in code),
plus per-severity counts on `import_batch`.

**Design the error record shape to be the shape the exception queue
(VS-013) will consume.** Retrofitting a common shape after four detectors
already emit their own formats is the expensive path — this is risk R-18.

## Expected API capabilities

Batch error list with filters (rule, severity, column). Error export to
CSV. Reprocess batch. Per-row accept/skip where a rule permits it.
Validation summary for a batch.

## Expected frontend screens

| Screen | Route |
|---|---|
| Batch errors tab, grouped counts | `/imports/$batchId/errors` |
| Row-level error table with filters | same |
| "Fix and reprocess" action | in-page |
| Downloadable error report | in-page |

Error text follows [`../../product/UX-PRINCIPLES.md`](../../product/UX-PRINCIPLES.md) §3:
name the row, the column, the business problem, and the next action. Never
a constraint name or a stack trace, and never a personal-data value.

## Acceptance criteria

1. A file of 100 rows with 3 bad rows imports 97 and reports 3 — it does
   not fail wholesale.
2. Every error names the row number, the column and the rule code.
3. After fixing a SKU mapping, reprocessing clears the corresponding errors
   and does **not** duplicate the 97 good rows.
4. Blocking-severity errors never reach business tables.
5. The data-quality rate KPI (valid rows ÷ total rows) is computable per
   batch and matches the displayed counts.
6. Error messages contain no personal-data values.

## Test layers

- **Unit** — every rule, at boundary values; severity classification;
  data-quality-rate calculation.
- **Integration** — mixed-validity file; reprocess idempotency at row level;
  blocking vs. warning behaviour; role matrix.
- **Contract** — client regenerated.
- **Frontend** — errors tab with many errors, with none, and while loading.
- **E2E** — import a deliberately broken fixture, see errors, fix the
  cause, reprocess, see them clear.

## Dependencies

VS-004. VS-003 for the unmapped-SKU rule.

## Risks

| Risk | Mitigation |
|---|---|
| Rule catalogue grows ad hoc during coding (R-01) | Catalogue with codes and severities written before implementation |
| Reprocessing double-counts good rows | Row-level idempotency, not only file-level; explicit test |
| Error shape diverges from what VS-013 needs (R-18) | Agree the shared shape here, in this slice |

## CV evidence

A validation rule catalogue with codes and severities; partial-success
semantics designed for a real operator; the data-quality KPI made
computable rather than aspirational.

## Explicit exclusions

No auto-correction of values. No ML anomaly detection. No exception
workflow, assignment or resolution states (VS-013). No notifications or
alerting. No order creation (VS-007).
