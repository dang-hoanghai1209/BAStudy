# Domain Glossary

**Status:** Draft skeleton (VS-000). English is the primary language of
this document; the Vietnamese Meaning column is a convenience reference,
not a substitute for the English definition. Terms will grow as slices
are delivered — treat this as living, not final.

| Term | English Definition | Vietnamese Meaning |
|---|---|---|
| Gross Revenue | Total sale value of an order before any deductions | Doanh thu gộp |
| Net Revenue | Revenue after discounts, refunds, and marketplace deductions | Doanh thu thuần |
| COGS | Cost of Goods Sold — the direct cost of the product(s) in an order | Giá vốn hàng bán |
| Marketplace Fee | Fee charged by the sales channel (commission, payment processing, etc.) | Phí sàn |
| Voucher | A discount instrument; may be funded by the seller or the marketplace | Voucher / mã giảm giá |
| Refund | Money returned to a buyer, in full or in part | Hoàn tiền |
| Reconciliation | Matching expected payout (calculated) against actual payout (received) | Đối soát |
| Order Profit | Net revenue minus COGS, fees, shipping, and allocated advertising cost for one order | Lợi nhuận của đơn hàng |
| Internal SKU | SellerMargin's own canonical product identifier, independent of any channel | Mã SKU nội bộ |
| Channel SKU | The product identifier as it exists on a specific marketplace | Mã SKU trên từng sàn |
| SKU Mapping | The link between a Channel SKU and an Internal SKU | Ánh xạ SKU |
| Cost History | A time-versioned record of what a SKU's cost was at each point in time | Lịch sử giá vốn |
| Import Batch | One file-upload event and everything that resulted from processing it | Lô dữ liệu import |
| Settlement / Statement | A marketplace's periodic financial report of what it paid or will pay the seller | Bảng sao kê / đối soát thanh toán |
| Payout | The actual money transferred from marketplace to seller | Tiền thực trả (payout) |
| Idempotent Import | Re-importing the same source data does not create duplicate records | Import không tạo dữ liệu trùng |
| Traceability | The ability to follow any calculated number back to its source data and the rule that produced it | Khả năng truy vết |

## Maintenance rule

When a vertical slice introduces a new business term or entity, add it
here in the same PR — don't let the glossary lag behind the data model.
