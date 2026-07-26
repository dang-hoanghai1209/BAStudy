# VS-018 — Reports and Exports

**Slice ID:** VS-018 · **Depends on:** VS-012, VS-014, VS-016, VS-017 · **Status:** Not started · **Level:** Seeded

> **Seeded spec.** Business analysis must be completed before Ready.

## Business objective

Let the accountant and the owner take the numbers out of the system in a
form their existing process actually accepts.

## BA learning outcome

Designing for the **downstream consumer**: the report is the handoff, so
its format is a requirement to be elicited, not a preference to be assumed.

## Business rules required — **pending**

| Question | Status |
|---|---|
| The report catalogue, and each report's exact definition | **Open** |
| Export formats — CSV first; anything else? | **Open** |
| Are exports point-in-time snapshots or live? | **Open** |
| Rounding and currency formatting in exports | Decided — must match on-screen exactly (ADR-0003) |
| **P-5** — who may export data containing personal data | **Open** |
| Is the Accountant a distinct role? | **Open** — shared with VS-016 |

## Entities involved

`report_definition` (optional), `export_job` if large exports need to run
asynchronously. Otherwise pure read paths over existing data.

## Expected API capabilities

Profit report per period, shop and SKU. Reconciliation report. Data-quality
report. Export any of these to CSV. Export job status where async.

## Expected frontend screens

Reports index (`/reports`); report viewer with parameters
(`/reports/$reportId`); export button with format and scope selection;
export history (`/reports/exports`).

## Acceptance criteria

1. **Exported figures equal the on-screen figures exactly**, including
   rounding — same query path, not a parallel implementation.
2. Every export is recorded in the audit trail with actor, scope and row
   count.
3. A large export does not block the request.
4. Exports respect tenant scope and role permissions.
5. Exports containing personal data are separately permissioned and emit
   `export.personal_data` with a mandatory reason.
6. CSV output is injection-safe: a cell beginning `=`, `+`, `-` or `@` is
   escaped so spreadsheet software does not execute it.

## Test layers

Unit (report aggregation; CSV formatting, quoting and injection escaping);
integration (parameterized reports; permission gating; export audit event
emitted; async job lifecycle); contract; frontend (report viewer states,
export progress); E2E (generate and download one report, compare to screen).

## Dependencies

VS-014 (aggregations), VS-012 (reconciliation data), VS-016 (permissions),
VS-017 (audit).

## Risks

| Risk | Mitigation |
|---|---|
| Export and screen computed by different code paths and disagreeing | Share the query layer; a test comparing both outputs for the same parameters |
| **CSV formula injection** — commonly missed in export features | Explicit escaping and a test with a malicious cell value |
| Personal data exported casually | Separate permission, separate audit event, mandatory reason |
| Report definitions invented rather than elicited | Ask the accountant persona what format they actually need |

## CV evidence

A stakeholder-driven deliverable with an elicited format specification;
export/screen equality proven by test; a security detail (CSV injection)
handled that most projects miss.

## Explicit exclusions

No PDF generation. No scheduled or emailed reports. No BI-tool connector.
No custom report builder. No accounting-package integration.
