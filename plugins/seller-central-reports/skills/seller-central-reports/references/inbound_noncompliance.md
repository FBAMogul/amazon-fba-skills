# Inbound Performance (`inbound_noncompliance`)

Problems Amazon found when receiving FBA shipments, such as missing or unscannable barcodes, missing prep and box quantity mismatches, one row per problem by product and shipment, with coaching level and any Unplanned Service fee; Amazon's SP-API page calls it the FBA Inbound Performance Report ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba); [Inbound problem summary](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200893760)).

## Where to find it in Seller Central

**Supply chain > Supply chain reports** (the Amazon Fulfilment Reports page), then **Inventory > Inbound Performance** in the left-hand list (`/reportcentral/INBOUND_NONCOMPLIANCE/1`). It sits near the end of the Inventory list, so click **Show more...** first ([Inbound Performance report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200875520)). Classic: **Reports > Fulfilment by Amazon**. Checked in Seller Central UK on 28 Oct 2026.

Amazon also flags problems in the Shipping Queue and by email (same help page). In the new menu, shipments are under **Supply chain > Manage shipments**; classic: **Inventory > Shipments** (in its FBA group).

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_FBA_FULFILLMENT_INBOUND_NONCOMPLIANCE_DATA`, `marketplaceIds` set to every store the account takes part in (for a UK account, its stores in Amazon's EU region), and `dataStartTime`/`dataEndTime` set to the window; then poll `getReport` until it's done and fetch the file with `getReportDocument` (unzip it if Amazon says GZIP). No report options. Amazon updates it daily ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). When a window has no problems, Amazon cancels the report (`CANCELLED`) rather than sending an empty file: it cancels a report automatically when there is no data to return ([getReport](https://developer-docs.amazon/sp-api/reference/getreport)).
- **Role:** [Amazon Fulfillment or Pricing](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types) (either is enough).
- **How far back Amazon goes:** not stated in Amazon's SP-API docs. The UK help page says the Inbound Performance report lets the seller generate or download problem reports for the past 365 days ([Inbound problem summary](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200893760)).

## Columns

Tab-separated text, header row first, 16 columns in this order.

| Column | Meaning | Format and traps |
|---|---|---|
| `issue-reported-date` | UK help page: "The date on which the delivery problem was resolved" | ISO 8601 with offset (`YYYY-MM-DDThh:mm:ss+00:00`). |
| `shipment-creation-date` | Date the shipment was created | ISO 8601 with offset. |
| `fba-shipment-id` | Shipment in which the problem was found | Starts `FBA`. |
| `fulfillment-center-id` | Fulfilment centre where the problem was found | Amazon's fulfilment centre code. |
| `sku` | Seller's SKU | Can be `--` even where `fnsku` and `asin` are filled. |
| `fnsku` | Amazon's fulfilment network SKU | |
| `asin` | Amazon product identifier | |
| `product-name` | Product title | |
| `problem-type` | Type of problem | An upper-case code with underscores, not the help page's wording. |
| `problem-quantity` | Units the problem applies to | |
| `coaching-level` | Coaching level for the problem group | Upper case, e.g. `STANDARD`. Repeated problems raise the coaching level ([Inbound Performance report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200875520)). Amazon names three levels: Standard, Elevated and Critical ([Performance coaching](https://sellercentral.amazon.com/gp/help/external/G5H26MCVJG4B82ST)); the others are probably `ELEVATED` and `CRITICAL`, but that isn't confirmed, so match without regard to case. |
| `fee-type` | Fee status of the problem | Blank when there's no fee. |
| `currency` | Not confirmed | Blank when there's no fee. |
| `fee-total` | Total Unplanned Service fees charged | Blank when there's no fee. |
| `problem-level` | Level of the problem | e.g. `ITEM`. |
| `alert-status` | Whether the problem has been acknowledged | e.g. `ACKNOWLEDGED`. |

Documented but not in the file: `fba-carton-id`, `expected-quantity`, `received-quantity`, `performance-measurement-unit`.

## Personal data

None of the columns hold buyer or business personal details. Some sellers put their cost price in the SKU, so treat SKUs as private.

## Questions it answers

- Which shipments had receiving problems, and of which types?
- What Unplanned Service fees were charged each month, and for which problems?
- Is any problem group at Elevated or Critical coaching level?
- Which SKUs keep causing labelling or prep problems?
- Which problems involved how many units arrived (`problem-type` with `problem-quantity`), to check against receipts in [`ledger_detail`](ledger_detail.md) and [`reimbursements`](reimbursements.md) (`reason` `Lost_Inbound`)? The report has no expected or received quantities.

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **A month with no problems comes back `CANCELLED`.** Amazon cancels the report when the window has no data, so a cancelled report here usually means no problems that month, not a failed request.
- **An empty or cancelled report doesn't prove no inbound fees were charged.** Cross-check fees in [`transactions`](transactions.md) or [`seller_economics`](seller_economics.md) before relying on it.
- **Formats not fully confirmed:** expect values not listed here.
- **`sku` can be `--`** while `fnsku` and `asin` are filled.
- **Four columns from Amazon's list are missing** from the file (above).
- Lines end in CRLF, but the last line of the file has no line ending at all, so `wc -l` counts one row short: a file with a single problem reads as 0 rows. The text is UTF-8.

Sources: [FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba), [Inbound Performance report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200875520).
