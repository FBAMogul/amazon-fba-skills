# Recommended Removals (`recommended_removals`)

Amazon's "FBA Recommended Removal Report": aged and unsellable FBA units per SKU, with the sellable units Amazon recommends removing before the next Inventory Cleanup, as of the report's `snapshot-date`. Amazon's API docs say it covers sellable items that will be 365 days old or older at the next Long-Term Storage cleanup, if run within six weeks of it, includes sellable and unsellable items, and is updated daily ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). The UK help page says it lists units that may be charged the long-term storage fee at the next Inventory Cleanup, on the 15th of each month, and works out per ASIN how many to remove to avoid it, assuming no more sales ([Recommended Removal Report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200688550)).

## Where to find it in Seller Central

Not in the Supply chain reports list when checked in Seller Central UK on 28 Oct 2026. That list (**Supply chain > Supply chain reports**, the Amazon Fulfilment Reports page; classic: **Reports > Fulfilment by Amazon**; [Amazon Fulfilment reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201104750)) had only **Removal Order Detail** and **Removal Shipment Detail** under **Removals**, and no Recommended Removals entry anywhere in the list. Amazon's [Inventory reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200885390) and [Removal Reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200989090) help pages still link it at `https://sellercentral.amazon.co.uk/gp/ssof/reports/search.html/?recordType=RECOMMENDED_REMOVAL`; whether that address still opens it is not confirmed.

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_FBA_RECOMMENDED_REMOVAL_DATA` and the marketplace ids of every store the account sells in (from Sellers `getMarketplaceParticipations`, where `isParticipating` is true and `domainName` starts `www.amazon.`, which leaves out the Non-Amazon entries), with no `dataStartTime` or `dataEndTime` and no report options; then poll `getReport` until it's done and fetch the file with `getReportDocument`, unzipping it if Amazon says GZIP. Amazon's [Request a report](https://developer-docs.amazon/sp-api/docs/request-a-report) page says the file covers every store sent unless the report type says otherwise; the [FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba) page says nothing otherwise for this one.
- **Role:** Amazon Fulfillment or Pricing for the report, either one is enough ([Role mappings for types](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)). The store list from `getMarketplaceParticipations` needs Product Listing or Selling Partner Insights in the EU ([role mappings](https://developer-docs.amazon/sp-api/docs/role-mappings)); without either, send the marketplace ids yourself.
- **How far back Amazon goes:** not stated in Amazon's docs. The report is current as of its `snapshot-date`, with no date range to ask for ([Recommended Removal Report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200688550)). Amazon can't make one for a past date, so history starts with the first copy you save. A copy your app requested in the last 90 days can still be fetched (`getReports`, then `getReportDocument`).

## Columns

Checked against a real UK seller's file in Oct 2026: 16 columns, in the order below. Tab-separated text, header row first.

| Column | Meaning | Format and traps |
|---|---|---|
| `snapshot-date` | The date the report's data is current as of. | ISO 8601 date-time with a time-zone offset. **Can be months old:** check it before reading the report as current (see Traps). |
| `sku` | Your SKU. | `£` comes as `&pound;`. |
| `fnsku` | Amazon's identifier for the unit stored in and fulfilled from its fulfilment centres. | |
| `asin` | Amazon's 10-character product identifier. | |
| `product-name` | The product title. | |
| `condition` | The product's condition. | A condition such as `New`, or `No Listing`. |
| `sellable-quantity` | Sellable units of the SKU in the fulfilment network. | Whole number. |
| `sellable-271-365-days` | Sellable units held for 271 to 365 days. | Comes before the younger bands. |
| `sellable-365+-days` | Sellable units held for 365+ days. | |
| `sellable-removal-quantity` | Sellable units to remove, assuming no more sales, to avoid the long-term storage fee at the next Inventory Cleanup. | Whole number. |
| `unsellable-quantity` | Unsellable units of the SKU in the fulfilment network. | |
| `unsellable-0-7-days` | Units unsellable for 0 to 7 days. | |
| `unsellable-8-60-days` | Units unsellable for 8 to 60 days. | |
| `unsellable-61-90-days` | Units unsellable for 61 to 90 days. | |
| `sellable-121-180-days` | Sellable units held in fulfilment centres for 121 to 180 days. | Second-to-last column. |
| `sellable-181-270-days` | Sellable units held for 181 to 270 days. | Last column. |

Every column Amazon lists is in the file, but in a different order: the 121-180 and 181-270 bands come last.

## Personal data

None: no buyer or business details. Some sellers put their cost price in the SKU, so treat SKUs as private. Rule: strip personal data before analysis and never copy it into anything shared.

## Questions it answers

- Which SKUs have units Amazon recommends removing before the next Inventory Cleanup, and how many?
- How much sellable stock sits in each age band (121-180, 181-270, 271-365, 365+ days)?
- How many units are unsellable, and for how long (0-7, 8-60, 61-90 days)? For unsellable stock now, use `inventory_all`'s `afn-unsellable-quantity`, plus `inventory_planning`'s `unfulfillable-quantity` for FNSKUs missing from `inventory_all` (FNSKUs holding only unsellable units can be missing from it). Count each FNSKU once.
- Which SKUs should go on a removal or disposal order? Only if `snapshot-date` is current; if it's stale, use `inventory_planning`.

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Can be stale.** `snapshot-date` can be months old on every row, though Amazon's docs say the report is updated daily. Don't read it as current without checking `snapshot-date`; use `inventory_planning` for today's ages and removal recommendations.
- **Columns out of order:** the `sellable-121-180-days` and `sellable-181-270-days` bands are last.
- No country column. `£` in SKUs comes as `&pound;`.
- Lines end in CRLF, but the last line of the file has no line ending at all, so `wc -l` counts one row short. The text is UTF-8.

Sources: [FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba) (column list), [Recommended Removal Report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200688550) (meanings), [US Recommended Removal Report](https://sellercentral.amazon.com/help/hub/reference/external/G200688550) (unsellable age bands, API names).
