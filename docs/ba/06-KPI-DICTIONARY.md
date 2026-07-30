# KPI Dictionary

**Status:** Draft skeleton (VS-000). Formulas here are placeholders
until `07-PROFIT-CALCULATION-SPEC.md` is finalized — treat as directional.

| KPI | Definition | Formula (draft) | Notes |
|---|---|---|---|
| Gross Revenue | Total order value before deductions | Sum of order line item prices × quantity | Per order/SKU/shop/period |
| Net Revenue | Revenue after discounts and refunds | Gross Revenue − Discounts − Refunds | |
| Order Profit | True profit for one order | Net Revenue − COGS − Marketplace Fees − Shipping − Allocated Advertising | See `07-PROFIT-CALCULATION-SPEC.md` for the authoritative formula |
| Margin % | Profit as a percentage of revenue | Order Profit ÷ Net Revenue | |
| Reconciliation Variance | Difference between expected and actual payout | Actual Payout − Expected Payout | Sign convention: positive = seller received more than expected |
| Data Quality Rate | Share of imported rows that passed validation without error | Valid Rows ÷ Total Rows in Import Batch | Per import batch |
| Unmapped SKU Rate | Share of channel SKUs without an internal SKU mapping | Unmapped Channel SKUs ÷ Total Channel SKUs | Should trend toward 0 |

## Maintenance rule

Any KPI shown on a dashboard screen must have an entry here with a
formula that matches the implementation exactly — if the formula
changes, this file and `07-PROFIT-CALCULATION-SPEC.md` update in the
same PR as the code.
