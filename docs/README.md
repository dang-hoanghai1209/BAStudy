# SellerMargin Documentation

This folder is the written half of SellerMargin. It is maintained in the
same pull request as the code it describes — a document that lags behind
the implementation is a defect, not a backlog item.

Entry point for anyone (human or agent) working in this repository:
[`../CLAUDE.md`](../CLAUDE.md).

## Reading order for someone new

1. **Why this exists** — [`ba/00-PROBLEM-STATEMENT.md`](ba/00-PROBLEM-STATEMENT.md),
   [`ba/01-PRODUCT-VISION.md`](ba/01-PRODUCT-VISION.md)
2. **Who it is for** — [`ba/02-STAKEHOLDER-MAP.md`](ba/02-STAKEHOLDER-MAP.md),
   [`product/PERSONAS.md`](product/PERSONAS.md)
3. **The vocabulary** — [`ba/03-DOMAIN-GLOSSARY.md`](ba/03-DOMAIN-GLOSSARY.md)
4. **What the repository actually is today** —
   [`architecture/CURRENT-STATE.md`](architecture/CURRENT-STATE.md)
5. **Where it is going** —
   [`architecture/TARGET-ARCHITECTURE.md`](architecture/TARGET-ARCHITECTURE.md),
   [`delivery/VERTICAL-SLICE-ROADMAP.md`](delivery/VERTICAL-SLICE-ROADMAP.md)
6. **The rules that govern the numbers** —
   [`ba/08-BUSINESS-RULES.md`](ba/08-BUSINESS-RULES.md)

## Folder map

| Folder | Holds | Changes when |
|---|---|---|
| `ba/` | Business analysis: problem, vision, stakeholders, glossary, scope, KPIs, calculation spec, business rules, UAT strategy | A business rule, KPI or scope boundary changes |
| `product/` | Personas, user journeys, UX principles, screen inventory | A user-facing flow or screen is added or reshaped |
| `architecture/` | Current state, target architecture, data model, API contract policy, generated-file rules, historical-data policy, ADRs | A structural or technical decision is made |
| `security/` | Authorization policy, audit policy, data-privacy policy | A permission, auditable action or data-handling rule changes |
| `delivery/` | Roadmap, task board, slice specs, DoR/DoD, test strategy, migration/rollback, baseline verification, risk register, templates | A slice starts, moves or finishes |
| `prompts/` | Reusable prompt library for implementation work | A recurring task shape is identified or improved |

## Source-of-truth hierarchy

Stated in full in [`../CLAUDE.md`](../CLAUDE.md) §2. In short, highest wins:

1. An explicit decision by the human owner (recorded in an ADR or a merged PR)
2. [`ba/08-BUSINESS-RULES.md`](ba/08-BUSINESS-RULES.md) — business and calculation questions
3. [`architecture/DATA-MODEL.md`](architecture/DATA-MODEL.md) — entity and schema questions
4. [`architecture/API-CONTRACT-POLICY.md`](architecture/API-CONTRACT-POLICY.md) plus the
   generated OpenAPI schema — API shape questions
5. [`security/AUTHORIZATION-POLICY.md`](security/AUTHORIZATION-POLICY.md) — who may do what
6. [`delivery/VERTICAL-SLICE-ROADMAP.md`](delivery/VERTICAL-SLICE-ROADMAP.md) — sequencing
7. Everything else is supporting detail, not a tie-breaker

## Current-state vs. target-state

These are deliberately two separate documents. `CURRENT-STATE.md` records
only what was verified to exist in this repository by direct inspection.
`TARGET-ARCHITECTURE.md` records what SellerMargin is being built toward.
Nothing may move from target to current without evidence from the repo.

## Document status labels

Every document carries a status in its header:

- **Verified** — asserted from direct repository inspection, with the evidence named
- **Decided** — a human decision recorded in an ADR; binding until superseded
- **Draft** — written but not yet validated against reality or with the owner
- **Open** — a placeholder that explicitly still needs an answer

There are no `[VERIFY]` tags left in this document set. Every one raised
during the VS-000 audit was either confirmed against the repository or
corrected. If a new assumption is introduced later, mark it **Open** and
give it an owner — do not reintroduce a silent one.
