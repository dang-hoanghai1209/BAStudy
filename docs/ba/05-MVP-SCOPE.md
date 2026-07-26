# MVP Scope

**Status:** Draft skeleton (VS-000). Scope reflects prior research;
confirm against `docs/delivery/TASK-BOARD.md` before treating as locked.

## In scope for MVP

- Login / authentication
- Organization and shop management
- Products and SKUs
- Cost of goods (with history, not a single overwritable field)
- CSV order import
- Import data validation
- Profit calculation
- Basic dashboard
- Import history

## Explicitly not in MVP

- Marketplace API integration (CSV only, for now)
- AI features of any kind
- Mobile app
- ERP integration
- Microservices / distributed architecture

## MoSCoW snapshot (initial)

| Priority | Item |
|---|---|
| Must | Org/shop context, Internal SKU + mapping, CSV import + validation, Cost history, Profit calculation, Dashboard |
| Should | Data quality alerts, Refund/return handling, Audit log |
| Could | Indirect expense allocation, Pricing/promotion simulator |
| Won't (this phase) | Marketplace API sync, AI insight assistant, FIFO inventory costing |

Full priority table with difficulty ratings lives in
`docs/delivery/VERTICAL-SLICE-ROADMAP.md`.

## Scope-creep guardrail

If a request during implementation would add a Should/Could/Won't item
into a Must slice, that's a stop condition — flag it per
[`../../CLAUDE.md`](../../CLAUDE.md) §10 rather than absorbing it silently
into the current slice. Each slice spec's "Explicit exclusions" section is
the concrete form of this guardrail.

## Context diagram (textual, pending visual)

```
[Seller / Analyst] --uses--> [SellerMargin]
[SellerMargin] --imports from--> [CSV export from Marketplace]
[SellerMargin] --produces--> [Dashboard, Reports, Alerts]
(Future) [SellerMargin] <--syncs with--> [Marketplace API]
```
