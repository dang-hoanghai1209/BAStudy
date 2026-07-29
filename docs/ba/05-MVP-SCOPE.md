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

## Marketplace scope

**Decided 2026-07-26 (human owner):** the MVP targets **Shopee and TikTok
Shop**; Lazada is deferred. Two channels is the minimum that proves the
cross-channel SKU rollup the product exists for. No real sample exports
are currently available — development proceeds on clearly-labelled
synthetic fixtures per
[ADR-0010](../architecture/adr/ADR-0010-MARKETPLACE-SCOPE-AND-TEST-DATA.md),
with the divergence exposure tracked as risk R-26.

## Explicitly not in MVP

- Lazada (and all other marketplaces beyond Shopee and TikTok Shop)
- Marketplace API integration (CSV only, for now)
- Multi-currency ([ADR-0003](../architecture/adr/ADR-0003-MONEY-AND-CURRENCY.md))
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
