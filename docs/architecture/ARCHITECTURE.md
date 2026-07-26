# Architecture — Index

**Status:** Index. This document used to hold both the current state and
the target design in one place, with unverified assumptions mixed into
both. The VS-000 audit split it, because conflating "what we have" with
"what we want" is how documentation starts lying.

## Read instead

| Question | Document |
|---|---|
| What does the repository actually contain today? | [`CURRENT-STATE.md`](CURRENT-STATE.md) — Verified |
| What are we building toward? | [`TARGET-ARCHITECTURE.md`](TARGET-ARCHITECTURE.md) |
| What entities exist, and what are they for? | [`DATA-MODEL.md`](DATA-MODEL.md) |
| How does the API contract stay honest? | [`API-CONTRACT-POLICY.md`](API-CONTRACT-POLICY.md) |
| Which files are generated, and how? | [`GENERATED-FILES.md`](GENERATED-FILES.md) |
| How is history protected from being rewritten? | [`HISTORICAL-DATA-POLICY.md`](HISTORICAL-DATA-POLICY.md) |
| Why is it built this way? | [`adr/`](adr/) |

## Decision record index

| ADR | Decision |
|---|---|
| [ADR-0001](adr/ADR-0001-MODULAR-MONOLITH.md) | Modular monolith, not microservices |
| [ADR-0002](adr/ADR-0002-TENANCY-MODEL.md) | `organization_id` everywhere; `shop_id` where shop-specific |
| [ADR-0003](adr/ADR-0003-MONEY-AND-CURRENCY.md) | `NUMERIC(18,4)` / `Decimal`; single-currency MVP |
| [ADR-0004](adr/ADR-0004-TEMPLATE-INHERITANCE.md) | What is kept, replaced and retired from the template |
| [ADR-0005](adr/ADR-0005-UPSTREAM-TRACKING.md) | Upstream tracked via git remote, not Copier |
| [ADR-0006](adr/ADR-0006-AUTHORIZATION-MODEL.md) | Platform superuser and organization Owner stay separate |
| [ADR-0007](adr/ADR-0007-FRONTEND-TEST-RUNNER.md) | Vitest + Testing Library, in its own commit |
| [ADR-0008](adr/ADR-0008-DEPLOYMENT-WORKFLOWS.md) | Inherited deploy workflows disabled |
| [ADR-0009](adr/ADR-0009-BACKEND-MODULE-STRUCTURE.md) | Flat until VS-005a, then domain packages |

New decisions use [`adr/ADR-TEMPLATE.md`](adr/ADR-TEMPLATE.md) and are
numbered sequentially. An ADR is superseded, never edited into a different
decision — the record of what was decided and why is the point.

## The short version

One FastAPI application, one PostgreSQL database, one React SPA, organised
internally by domain. Four data layers — source, canonical, business,
calculated — so a wrong number can be isolated to a layer. Every business
table is organization-scoped. Money is exact decimal. Nothing with history
is ever overwritten.
