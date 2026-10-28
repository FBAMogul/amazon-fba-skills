# Removal Order Detail (`removal_orders`)

FBA removal and disposal orders placed during the period, one row per FNSKU per order, with status, the quantities requested, shipped, disposed, cancelled and in process, and the removal fee; it covers returns, disposals and liquidations ([Removal Order Detail report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200989110)).

## Where to find it in Seller Central

**Supply chain > Supply chain reports** (the Amazon Fulfilment Reports page), then **Removals > Removal Order Detail** in the left-hand list. Classic: **Reports > Fulfilment by Amazon**. Checked in Seller Central UK on 28 Oct 2026. Amazon's help groups it with the removal reports ([Removal Reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200989090)) and links it at `sellercentral.amazon.co.uk/reportcentral/REMOVAL_ORDER_DETAIL/0` ([Removal Order Detail report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200989110)).

The online view shows one row per order; the download, and the API report, one row per FNSKU.

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_FBA_FULFILLMENT_REMOVAL_ORDER_DETAIL_DATA`, `marketplaceIds` set to every store the account takes part in (for a UK account, its stores in Amazon's EU region), and `dataStartTime`/`dataEndTime` set to the window; then poll `getReport` until it's done and fetch the file with `getReportDocument` (unzip it if Amazon says GZIP). No report options. Amazon updates it in near real time ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). Amazon can throttle `createReport` for this report type (see Traps seen in real data).
- **Role:** [Amazon Fulfillment or Pricing](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types) (either is enough).
- **How far back Amazon goes:** not stated in Amazon's docs.

## Columns

Tab-separated text, header row first, 17 columns in this order.

| Column | Meaning | Format and traps |
|---|---|---|
| `request-date` | Date the removal order was submitted | ISO 8601, but the time zone offset varies from row to row. Convert to UTC before comparing. |
| `order-id` | Unique ID for the removal order | 9 or 10 characters: letters and digits, sometimes with `/` or `+`. |
| `order-source` | Not defined in Amazon's docs; who or what created the order | Text naming the source: a removal the seller asked for by hand, or an automatic removal, set up by the seller or made by Amazon. |
| `order-type` | Removal order type: return, disposal or liquidations | Text, e.g. `Return` or `Disposal`. |
| `service-speed` | Not defined in Amazon's docs | `Standard`, or `-` on disposals. Appears once, not twice as Amazon's list has it. |
| `order-status` | Removal order status | Text, e.g. `Completed` or `Cancelled`. The wording of an open status isn't confirmed. |
| `last-updated-date` | Date of the most recent order update | Same varying offsets, `+00:00` included. Can be weeks or months after `request-date` (see Traps seen in real data). |
| `sku` | Seller's SKU | Can be an `amzn.gr.…` (Amazon Grade and Resell) SKU. **HTML entities:** `£` comes as `&pound;`, `"` as `&quot;`. Decode every entity before joining to other files; replacing `&pound;` alone still leaves some SKUs that won't match. |
| `fnsku` | Amazon's fulfilment network SKU | |
| `disposition` | Unit condition | `Unsellable` or `Sellable`. |
| `requested-quantity` | Units of this FNSKU requested in the order | |
| `cancelled-quantity` | Units of this FNSKU cancelled from the order | |
| `disposed-quantity` | Units of this FNSKU disposed of | Blank except on disposal lines. |
| `shipped-quantity` | Units of this FNSKU delivered for this return | |
| `in-process-quantity` | Units of this FNSKU still in process | `0` once an order is closed. |
| `removal-fee` | Total removal fee for all units of this FNSKU | **Includes VAT.** Added up per order, it equals minus the `totalAmount` of the [`transactions`](transactions.md) charges with that `ORDER_ID`, not minus their `Base`: about 1.2 times the fee before VAT. Blank on every cancelled line and on some completed ones. Often charged in a later month than the order was requested in: don't total it by month (see Questions it answers). |
| `currency` | Currency of the removal fee | e.g. `GBP`. Blank when `removal-fee` is. |

## Personal data

None of the columns hold buyer or business personal details, and the report has no address columns. Some sellers put their cost price in the SKU, so treat SKUs as private.

## Questions it answers

- Do units requested for removal match units actually removed? For each order, do shipped, disposed, cancelled and in-process units add up to what was requested?
- Which removal or disposal orders are still open, and how many units are in process? An order can still change weeks or months after it was requested, so use a recently fetched copy of the report.
- What did removals and disposals cost each month? Not from this report. Use [`transactions`](transactions.md): the `ServiceFee` lines whose `description` is `FBARemoval` or `FBADisposal`, by `postedDate`, totalled per `totalAmount.currencyCode`.
  - `totalAmount` includes VAT, like `removal-fee`, while [`storage_fees`](storage_fees.md) and [`long_term_storage_fees`](long_term_storage_fees.md) give fees before VAT, so put them on the same basis before adding them up. Which basis is the cost depends on the seller's VAT position (see [SKILL.md's VAT section](../SKILL.md#vat-included-or-not-per-file)): a seller who reclaims fee VAT (VAT-registered on standard accounting) counts fees before VAT; a Flat Rate Scheme or non-registered seller can't reclaim it, so counts fees with VAT.
  - For fees before VAT, add up each item's `Base` and `Promo` (`Base` is the charge, negative; `Promo` is a discount, positive; `Tax` is the VAT). `Base` + `Tax` + `Promo` equals the item's `totalAmount`. Lines usually carry `Tax`; on some, `Promo` cancels `Base`, so they total 0.
  - The fee sits under `AmazonFees > FBARemovalFee` or `FBADisposalFee`, and on some lines from June 2026 under `FBAFees > FBARemovalFee` or `FBADisposalFee`: take every `Base` and `Promo`, whatever the path.
  - A report for one month holds the orders requested in that month (UTC), and their fees are often charged in a later month, so a month's `removal-fee` total rarely equals that month's charges.
  - Some `FBADisposal` charges carry a customer order id (`123-1234567-1234567` form) and are in neither removal report.
- Which SKUs were returned and which disposed of, and in what condition?

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Throttled when many months were requested.** After a handful of quick requests (one per month), Amazon refused `createReport` for this report type with HTTP 429 while still accepting other report types. It let one more through a little later, then refused the rest for about an hour. Later runs fetched the remaining months a few at a time before being refused again. Amazon doesn't document this.
- **Late updates.** Rows can be last updated, and get their fee, weeks or even months after the month they were requested in. A copy fetched soon after a month ends keeps the older status and may have no fee, so fetch recent months again, and take fees from [`transactions`](transactions.md) (above). [`removal_shipments`](removal_shipments.md) is the same.
- **Orders can be automatic.** As well as removals the seller asks for by hand, lines can come from automatic removals, set up by the seller or made by Amazon; `order-source` says which.
- **`removal-fee` is blank** on every cancelled line and on some completed ones.
- **Date offsets vary by row.** Convert to UTC.
- Lines end in CRLF, but the last line of every file has no line ending at all, so `wc -l` counts one row short. The text is UTF-8.

Sources: [FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba), [Removal Order Detail report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200989110).
