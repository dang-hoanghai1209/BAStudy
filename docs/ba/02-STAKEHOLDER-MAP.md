# Stakeholder Map

**Status:** Draft skeleton (VS-000). Roles below are hypothesized from
the product vision, not yet confirmed by interviews.

## Primary users

| Stakeholder | Role in system | Primary needs |
|---|---|---|
| Shop Owner | Data owner, decision-maker | Trustworthy profit numbers, simple dashboard |
| Seller Manager / Analyst | Day-to-day operator | Import data, manage costs/fees, investigate anomalies |
| Accountant / Bookkeeper | Downstream consumer | Reconciliation reports, exportable statements |

## Secondary / system stakeholders

| Stakeholder | Interest |
|---|---|
| Marketplace platforms (Shopee, TikTok Shop, Lazada, etc.) | Source of order/fee/statement data — not a direct user of the system |
| Future integration developer | Consumes the marketplace adapter interface |
| Portfolio reviewer / interviewer (project's secondary goal) | Reads BA documentation and demo as evidence of BA skill |

## Roles inside the system (initial)

- **Owner** — highest role **within one organization**
- **Analyst** — day-to-day operational access (import, mapping, cost
  entry, viewing)
- **Viewer** — read-only

Separately, and deliberately **not** part of this hierarchy, the platform
retains `is_superuser` for administering the SellerMargin installation
itself. A superuser is not implicitly a member of any organization and gets
no access to organization business data. See
[`../architecture/adr/ADR-0006-AUTHORIZATION-MODEL.md`](../architecture/adr/ADR-0006-AUTHORIZATION-MODEL.md).

The detailed permission matrix lives in
[`../security/AUTHORIZATION-POLICY.md`](../security/AUTHORIZATION-POLICY.md).
This file only tracks who the roles represent.

Fuller persona detail — goals, pains, design implications — is in
[`../product/PERSONAS.md`](../product/PERSONAS.md).

## Open questions

- Who is authorized to edit an active fee rule or confirm a
  reconciliation? Mandatory question #15. The working answer in the
  authorization policy is **Owner only**, recorded as Draft pending
  confirmation.
- Will there be a distinct "Accountant" role with different permissions
  than Analyst, or is Accountant simply a Viewer with export rights?
  **Not yet decided** — resolved in VS-016 or VS-018, whichever comes
  first.
