# Vertical Slice Spec Template

Copy this into [`slices/`](slices/) for each new slice (e.g.
`VS-004-CSV-IMPORT-CENTER.md`) before any code is written for it.

A spec is at **Seeded** level while its business-analysis section still
contains open questions, and at **Specified** level once they are answered.
Only a Specified spec can pass the Definition of Ready.

---

**Slice ID:** VS-0xx
**Title:** <name>
**Depends on:** <prior slice IDs, per `VERTICAL-SLICE-ROADMAP.md`>
**Status:** Not started | Blocked | In progress | In review | Done
**Level:** Seeded | Specified

## 1. Business analysis

- Business objective — what the seller can do afterwards that they could
  not before
- BA learning outcome — what analytical skill this slice demonstrates
- User story / stories (As a ___, I want ___, so that ___)
- Acceptance criteria per story
- Relevant business rules (link to `../ba/08-BUSINESS-RULES.md` entries —
  add new ones there if this slice introduces them)
- Which mandatory questions **and D-series decisions** apply here, and
  their answers. Pending means **not ready**, per
  `DEFINITION-OF-READY-DONE.md`

## 2. Data model impact

- New/changed entities (link to `docs/architecture/DATA-MODEL.md`)
- Migration plan, including whether it's additive or requires care
  around existing data

## 3. API / OpenAPI contract

- New/changed endpoints, request/response shapes
- Any breaking change and why (see `API-CONTRACT-POLICY.md`)

## 4. Backend implementation notes

- Module(s) touched (per `docs/architecture/ARCHITECTURE.md`)
- Key logic/edge cases to handle

## 5. Generated client

- Confirm client regenerated after backend contract is stable — link
  the regeneration step, don't hand-edit

## 6. Frontend implementation notes

- Screen(s)/component(s) touched
- States covered: loading, empty, error, happy path

## 7. Tests

- Unit / integration / contract / frontend / E2E — map each to what
  it proves (see `TEST-STRATEGY.md`)

## 8. Traceability

- How does a value produced by this slice trace back to its source
  data and the rule that produced it?

## 9. UAT

- Scenario(s) walked through, actual vs. expected result (see
  `docs/ba/09-UAT-STRATEGY.md`)

## 10. Integration report

- Link the completed `INTEGRATION-REPORT-TEMPLATE.md` for this slice

## 11. Risks

- Slice-specific risks and their mitigations. Link to `RISK-REGISTER.md`
  for anything that is a standing risk rather than a slice-local one

## 12. CV evidence

- What this slice demonstrates to a reviewer — the artifact or proven
  behaviour, not the feature name

## 13. Explicit exclusions

- What this slice deliberately does **not** do, and which slice owns it
  instead. This section is binding: absorbing an excluded item is scope
  creep and a stop condition per `../../CLAUDE.md` §10
