# Reimbursements (`reimbursements`)

Every FBA inventory reimbursement line, paid as cash or as replacement units, with its reason, case ID and amounts, including reversals; it holds both the reimbursements the seller claimed and the ones Amazon made automatically ([Reimbursements report (overview)](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200732720)).

## Where to find it in Seller Central

**Supply chain > Supply chain reports** (the Amazon Fulfilment Reports page), then **Payments > Reimbursements** in the left-hand list (`/reportcentral/REIMBURSEMENTS/0`, which opens the online view). Classic: **Reports > Fulfilment by Amazon**. Checked in Seller Central UK on 28 Oct 2026. Amazon's help also lists it among its FBA payments reports ([Payments reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200885400)).

Reimbursements also show in the Transaction view, under Other, as Balance Adjustment or FBA Inventory Reimbursement ([Reimbursements report (overview)](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200732720)). In the new menu that is **Finance > Finance reports**, **Transaction View** tab.

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_FBA_REIMBURSEMENTS_DATA`, `marketplaceIds` set to every store the account takes part in (for a UK account, its stores in Amazon's EU region), and `dataStartTime`/`dataEndTime` set to the window; then poll `getReport` until it's done and fetch the file with `getReportDocument` (unzip it if Amazon says GZIP). No report options. Amazon updates it daily ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)).
- **Role:** [Amazon Fulfillment or Pricing](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types) (either is enough).
- **How far back Amazon goes:** not stated in Amazon's SP-API docs. The UK help page on reversals says the Seller Central report's search can be widened to the past 18 months ([Reversed Reimbursement](https://sellercentral.amazon.co.uk/help/hub/reference/external/GL9ZDED52WXZUMFD)).

## Columns

Tab-separated text, header row first, 18 columns in this order.

| Column | Meaning | Format and traps |
|---|---|---|
| `approval-date` | Date the reimbursement was approved | ISO 8601 with offset (`YYYY-MM-DDThh:mm:ss+00:00`). A reimbursement can take up to five days after approval to appear in the report ([Reimbursements report (overview)](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200732720)). |
| `reimbursement-id` | Unique ID for the reimbursement | Digits. One reimbursement may have several lines, and same-day reimbursements for one issue may be combined into one line ([Reimbursements report (overview)](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200732720)). |
| `case-id` | ID of the case, when the seller requested the reimbursement | Often blank: only filled when the seller opened a case. |
| `amazon-order-id` | Customer order ID | Blank when the line isn't tied to an order. |
| `reason` | Reason for the reimbursement | Codes, not the help page's wording: most with underscores, such as `Lost_Warehouse`, but some without (`CustomerReturn`). Reversals are `Reimbursement_Reversal`. |
| `sku` | Seller's SKU | **HTML entities:** `£` comes as `&pound;`; `&amp;` and `&quot;` occur too. Decode every entity, not just `&pound;`, before joining to other files. |
| `fnsku` | Amazon's fulfilment network SKU | |
| `asin` | Amazon product identifier | |
| `product-name` | Product name | |
| `condition` | Item condition | Condition codes such as `NewItem`. |
| `currency-unit` | Currency for the store | e.g. `GBP`. Lines for another store come in that store's currency (e.g. `EUR`), so total per currency. |
| `amount-per-unit` | Per-unit reimbursement amount for the line | Negative on reversals. |
| `amount-total` | Total cash reimbursed for the line | Negative on reversals, so summing it gives the net cash. `0` on lines reimbursed only with units. |
| `quantity-reimbursed-cash` | Units reimbursed with cash | Negative on reversals. |
| `quantity-reimbursed-inventory` | Units reimbursed with inventory | Usually `0`; can be blank (for example on reversals and `Damaged_Inbound` lines). **On reversals it's often a positive number** while the cash quantity is negative and the total `0`: that is not a replacement unit. |
| `quantity-reimbursed-total` | Cash units plus inventory units | `0` or negative on reversals. Don't sum it for a net count (see Questions it answers). |
| `original-reimbursement-id` | ID of the original reimbursement being adjusted | Only on adjustments. |
| `original-reimbursement-type` | Type of the original reimbursement being adjusted | Only on reversals. Holds the same codes as `reason` (e.g. `Lost_Warehouse`), not the help page's "Lost: Warehouse". |

Documented but not in the file: `quantity` (on the UK help page only).

## Personal data

None of the columns hold buyer or business personal details. Some sellers put their cost price in the SKU, so treat SKUs as private.

## Questions it answers

- How much did Amazon reimburse each month, and for which reasons? Total `amount-total` per `currency-unit`.
- Which reimbursements came as cash and which as replacement units? For net units, sum `quantity-reimbursed-cash` over every row (negative on reversals) and add `quantity-reimbursed-inventory` only on rows whose `reason` isn't `Reimbursement_Reversal` (see that column). Never use `quantity-reimbursed-total` for a net count: on reversals it can add that inventory figure back, so summing it overstates the units.
- Which reimbursements were later reversed, and against which original?
- Which lost or damaged units in [`ledger_detail`](ledger_detail.md) have no reimbursement yet (match on FNSKU)?
- Which of the seller's cases (`case-id`) paid out, and how much?

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **HTML entities in SKUs.** This report writes `£` as the HTML entity `&pound;`, and also writes `&amp;` and `&quot;`. Replacing `&pound;` alone leaves some SKUs unmatched against the plain-text reports ([`orders`](orders.md), [`listings_all`](listings_all.md), [`inventory_all`](inventory_all.md), [`ledger_detail`](ledger_detail.md), [`fba_shipments`](fba_shipments.md)); after decoding every entity, all match. Several other reports do the same.
- **Reasons are codes with underscores** (`Lost_Warehouse`), not the help page's wording. Reversals are lines of their own, with `reason` `Reimbursement_Reversal` and negative amounts.
- **`quantity-reimbursed-inventory` on reversals** is usually a positive number that isn't a replacement unit, so `quantity-reimbursed-total` can't be summed for net units (see Questions it answers).
- **`case-id` is often blank:** only filled when the seller opened a case.
- **Not throttled:** Amazon accepted many months requested one after another for this report.
- Lines end in CRLF, but the last line of each file has no line ending at all, so `wc -l` counts one row short and a shell `while read` loop skips that row. The text is UTF-8.

Sources: [FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba), [Reimbursements report (overview)](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200732720), [Reimbursement report definitions](https://sellercentral.amazon.co.uk/help/hub/reference/external/GMGJASYYNYLXKLXE), [Reversed Reimbursement](https://sellercentral.amazon.co.uk/help/hub/reference/external/GL9ZDED52WXZUMFD).
