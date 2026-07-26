# Business Analysis Documentation — SellerMargin

Business-analyst artifacts for SellerMargin, maintained alongside the code
rather than ahead of it or apart from it. Each vertical slice updates the
relevant file here as part of its Definition of Done.

## Reading order

1. [`00-PROBLEM-STATEMENT.md`](00-PROBLEM-STATEMENT.md) — why this exists
2. [`01-PRODUCT-VISION.md`](01-PRODUCT-VISION.md) — what SellerMargin is and is not
3. [`02-STAKEHOLDER-MAP.md`](02-STAKEHOLDER-MAP.md) — who it is for
4. [`03-DOMAIN-GLOSSARY.md`](03-DOMAIN-GLOSSARY.md) — shared vocabulary
5. [`04-AS-IS-TO-BE.md`](04-AS-IS-TO-BE.md) — the process being replaced
6. [`05-MVP-SCOPE.md`](05-MVP-SCOPE.md) — what ships first
7. [`06-KPI-DICTIONARY.md`](06-KPI-DICTIONARY.md) — how success is measured
8. [`07-PROFIT-CALCULATION-SPEC.md`](07-PROFIT-CALCULATION-SPEC.md) — the core calculation
9. [`08-BUSINESS-RULES.md`](08-BUSINESS-RULES.md) — the rules governing every calculation and workflow
10. [`09-UAT-STRATEGY.md`](09-UAT-STRATEGY.md) — how a slice gets proven, not just tested

Then continue into the neighbouring folders:
[`../product/`](../product/) turns these into personas, journeys and
screens; [`../architecture/`](../architecture/) turns them into entities and
contracts; [`../security/`](../security/) turns them into permissions and
policy.

## Status

Every file here is at **Draft** status. They carry the structure and the
content brought forward from prior research, plus explicit open questions —
not final answers. Each file states its own status in its header.

Two of these are not merely drafts but genuine gaps:

- [`00-PROBLEM-STATEMENT.md`](00-PROBLEM-STATEMENT.md) — **no seller
  interviews have been conducted.** Every pain point is an assumed pain
  point, and should be described that way until at least three interviews
  happen.
- [`08-BUSINESS-RULES.md`](08-BUSINESS-RULES.md) — the source of truth for
  business logic, currently holding **fifteen mandatory questions plus
  eight further open decisions**, and no answered rules. Several slices are
  blocked on it.

## Maintenance rule

A business rule, KPI or glossary term that changes in code changes here in
the same pull request. A rule that exists only in code is a defect — see
risk R-01 in [`../delivery/RISK-REGISTER.md`](../delivery/RISK-REGISTER.md).
