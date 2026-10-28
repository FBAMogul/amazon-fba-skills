# Manage FBA Inventory – Archived (`inventory_all`)

Amazon's "FBA Manage Inventory - Archived" report: FBA stock per SKU by status (sellable, unsellable, reserved, inbound, researching), archived listings included, with condition, quantity and volume, as it stands now and updated in near real-time ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). It is the source for UK stock ready to pick, but it leaves out stock held outside the UK, stranded stock and some UK FNSKUs holding only reserved or unsellable units. The same report without archived listings is `GET_FBA_MYI_UNSUPPRESSED_INVENTORY_DATA`.

## Where to find it in Seller Central

**Supply chain > Supply chain reports** (the Amazon Fulfilment Reports page; [Amazon Fulfilment reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201104750)), then **Inventory > Manage FBA Inventory Report - Archived** in the left-hand list (`/reportcentral/FBA_MYI_ALL_INVENTORY/1`; **Show more...** expands the Inventory list). **Manage FBA Inventory**, next to it (code `FBA_MYI_UNSUPPRESSED_INVENTORY`), is the same report without archived listings. Classic: **Reports > Fulfilment by Amazon**. Checked in Seller Central UK on 28 Oct 2026. Amazon's [Inventory reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200885390) help page lists it too.

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_FBA_MYI_ALL_INVENTORY_DATA` and the marketplace ids of every store the account sells in (from Sellers `getMarketplaceParticipations`, where `isParticipating` is true and `domainName` starts `www.amazon.`, which leaves out the Non-Amazon entries), with no `dataStartTime` or `dataEndTime` and no report options; then poll `getReport` until it's done and fetch the file with `getReportDocument`, unzipping it if Amazon says GZIP. Amazon's [Request a report](https://developer-docs.amazon/sp-api/docs/request-a-report) page says the file covers every store sent unless the report type says otherwise; the [FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba) page says nothing otherwise for this one.
- **Role:** Amazon Fulfillment or Pricing for the report, either one is enough ([Role mappings for types](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)). The store list from `getMarketplaceParticipations` needs Product Listing or Selling Partner Insights in the EU ([role mappings](https://developer-docs.amazon/sp-api/docs/role-mappings)); without either, send the marketplace ids yourself.
- **How far back Amazon goes:** not stated in Amazon's docs. The report shows stock as it is now, with no date range to ask for ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). Amazon can't make one for a past date, so history starts with the first copy you save. A copy your app requested in the last 90 days can still be fetched (`getReports`, then `getReportDocument`).

## Columns

Checked against a real UK seller's file in Oct 2026: 26 columns, in this order. Tab-separated text, header row first.

| Column | Meaning | Format and traps |
|---|---|---|
| `sku` | Your SKU. | One row per SKU; no country column. |
| `fnsku` | Amazon's identifier for the unit stored in and fulfilled from its fulfilment centres. | |
| `asin` | Amazon's 10-character product identifier. | |
| `product-name` | The product title. | |
| `condition` | The product's condition. | A condition code written as one word, such as `New`; can be `Unknown`. |
| `your-price` | Your current selling price. | Currency not stated; UK prices. |
| `mfn-listing-exists` | "Yes" if the product is listed for fulfilment by you. | `Yes` or `No`. |
| `mfn-fulfillable-quantity` | Units in your own fulfilment network that can be picked, packed and shipped. | Can be blank. |
| `afn-listing-exists` | "Yes" if the item is listed for Fulfilment by Amazon. | `Yes` or `No`. |
| `afn-warehouse-quantity` | Units in fulfilment centres that have been processed. | Whole number. |
| `afn-fulfillable-quantity` | Units in fulfilment centres that can be picked, packed and shipped. | Whole number. |
| `afn-unsellable-quantity` | Units in fulfilment centres in unsellable condition. | |
| `afn-reserved-quantity` | Units reserved for customer orders, staging, or FC processing. | |
| `afn-total-quantity` | Units in fulfilment centres or inbound. | **Includes inbound:** it equals `afn-warehouse-quantity` plus the three `afn-inbound-…` columns, and on some rows all of it is inbound. For units in the centres, read `afn-warehouse-quantity`. |
| `per-unit-volume` | Volume of one unit. | Decimal number. **Cubic centimetres**, not the cubic feet the help page says: against the cubic-foot `item_volume` in `storage_fees` for the same FNSKU, the median ratio is within 1 of the 28,317 cm³ in a cubic foot. Single SKUs vary widely. |
| `afn-inbound-working-quantity` | Units in a shipment you've told Amazon about. | |
| `afn-inbound-shipped-quantity` | Units in a shipment you've told Amazon about and given a tracking number for. | |
| `afn-inbound-receiving-quantity` | Units in a shipment that has arrived at a fulfilment centre but aren't processed yet. | |
| `afn-researching-quantity` | Units being checked for possible reimbursement after confirmed or suspected loss or damage. | |
| `afn-reserved-future-supply` | Units customers have bought while stock is on its way through In-Stock Head Start. | |
| `afn-future-supply-buyable` | Units customers can find and buy while stock is on its way through In-Stock Head Start. | Not always stock on its way: it can be above 0 on SKUs with no stock or inbound units at all (for example on `Inactive` and `Incomplete` listings), and on SKUs with stock in the centres and nothing inbound. `inventory_by_country` counts it only where units are inbound. |
| `afn-fulfillable-quantity-local` | Not in Amazon's docs; by name, fulfillable units in the local (UK) network. | Seen equal to `afn-fulfillable-quantity`. |
| `afn-fulfillable-quantity-remote` | Not in Amazon's docs; by name, fulfillable units held in other countries. | Can be `0` on every row, even where SKUs have stock in EU fulfilment centres: take stock in other countries from `inventory_by_country`. |
| `afn-fc-transfer-quantity` | Units moving between fulfilment centres. | |
| `afn-onhand-buyable-quantity` | Units customers can buy now: `afn-fulfillable-quantity` + `afn-fc-transfer-quantity`. | |
| `store` | The store the stock is sold through, e.g. Low-Cost Store or FBA from origin. | Empty means FBA only. |

Every column Amazon lists is in the file.

## Personal data

None: no buyer or business details. Some sellers put their cost price in the SKU, so treat SKUs as private. Rule: strip personal data before analysis and never copy it into anything shared.

## Questions it answers

- How many units of each SKU are ready to pick in the UK right now (`afn-fulfillable-quantity`), and how many are unsellable? Use this report for pickable UK stock, not `restock` or `inventory_planning`, which can lag it (see Traps). Don't add inbound from another report to this one's stock. It isn't all of FBA: stock in other countries is in `inventory_by_country`'s non-`GB` rows, plus any EU stock that only `storage_fees` bills (see `inventory_by_country`); stranded units are in `stranded` (and already in `inventory_by_country`'s `GB` rows, so count them once); UK FNSKUs holding only reserved or unsellable units are in `inventory_planning`.
- Which SKUs have stock inbound, and at which stage (working, shipped, receiving)?
- Which units are in researching status, and so worth watching for a reimbursement?
- Which SKUs still hold stock at Amazon though their listing isn't active? Read `afn-warehouse-quantity` above 0, not `afn-total-quantity` (some SKUs have only inbound stock). Add the stock this report leaves out (above): EU-held stock, stranded SKUs and FNSKUs with only reserved or only unsellable units. Then look each SKU, decoded, up in the `listings_all` file of the store that holds the stock, and count a SKU with no row there as not listed, so not active. Such SKUs can have no row in that store's `listings_all` file, so an inner join on `status` loses them. No column says whether a listing is archived.
- How did each SKU's stock change week on week, comparing saved snapshots?

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Old SKUs with no stock.** Archived listings are included, so many rows can be old SKUs with no stock or inbound units. No column says which listings are archived.
- **No country split, and stock left out.** Rows aren't split by store or country though the request names every store, and stock held outside the UK is missing: a SKU whose stock is held only in EU fulfilment centres can have no row here, or a row with no stock. `inventory_planning` can hold FNSKUs that aren't here, some with stock: stranded SKUs, FNSKUs with only reserved units and FNSKUs with only unsellable units. Use `inventory_by_country`'s non-`GB` rows for EU stock (plus any EU stock only `storage_fees` bills), and `inventory_planning` for those UK FNSKUs. Stranded units are in `stranded`, and already in `inventory_by_country`'s `GB` rows: count them once.
- **The other stock reports can lag this one.** `restock` and `inventory_planning`, fetched a few minutes after this report, can still show as inbound units this report already has as fulfillable; `ledger_detail` can show their receipt only the next day. Take pickable stock from here.
- **`per-unit-volume` is in cubic centimetres**, not cubic feet.
- Lines end in CRLF, but the last line of the file has no line ending at all, so `wc -l` counts one row short and a shell `while read` loop skips that row. The text is UTF-8.

Sources: [FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba) (column list), [Manage FBA Inventory report: Archive](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200740940) and [Manage FBA Inventory report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200740930) (meanings), [US Manage FBA Inventory report](https://sellercentral.amazon.com/help/hub/reference/external/G200740930) (`store`).
