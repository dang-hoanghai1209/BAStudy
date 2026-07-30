# Product Vision

**Status:** Draft skeleton (VS-000), carried over from prior research.
Should be reviewed by the human owner before being treated as final.

## Vision statement

**SellerMargin — E-commerce Profitability & Settlement Reconciliation
Platform.** A system that calculates true profit for multi-channel
sellers and reconciles expected payout against what marketplaces
actually pay out.

## Core value, in order

1. Ingest sales data from CSV (marketplace API integration later)
2. Normalize products and SKUs across multiple channels
3. Calculate real profit per order, per SKU, and per shop
4. Reconcile expected payout against actual marketplace payout
5. Let every number be traced back to its source row and the rule that
   produced it
6. Alert on missing data, incorrect cost data, or unreconciled amounts

## What SellerMargin is not

- Not a full ERP or inventory management system
- Not a fulfillment / shipping-label system
- Not a full HR, CRM, or supplier management system
- Not an official tax filing system
- Not a full double-entry accounting system
- Not built on microservices/Kafka/Kubernetes — see
  [`../architecture/adr/ADR-0001-MODULAR-MONOLITH.md`](../architecture/adr/ADR-0001-MODULAR-MONOLITH.md)
- Not an AI-forecasting product — AI features are deliberately deferred
  until the profit engine and underlying data are trustworthy (see
  §"AI boundaries" below)

This boundary exists so the project has real depth without exceeding
what a single-developer-plus-agent project can responsibly deliver.

## AI boundaries

AI assistance is appropriate for: explaining why margin changed,
summarizing weekly alerts, suggesting analysis questions, flagging
anomalous transactions, suggesting (not confirming) SKU mappings,
turning natural-language requests into dashboard filters, and drafting
fee-rule text from a description.

AI must never: change COGS on its own, merge SKUs on its own, confirm a
reconciliation on its own, edit an active fee rule on its own, present a
price decision without showing the underlying formula, or delete import
data. AI-facing answers must be grounded in stored data and the
calculation engine, never an estimate presented as a calculated result.

## Success looks like

See `06-KPI-DICTIONARY.md` for measurable definitions. Qualitatively:
a seller can open the dashboard, see per-order profit that matches what
actually lands in their bank account, and trust it enough to make
pricing and channel decisions from it.
