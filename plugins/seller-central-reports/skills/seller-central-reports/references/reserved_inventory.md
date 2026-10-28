# FBA Reserved Inventory (`reserved_inventory`)

Amazon's "FBA Reserved Inventory Report": FBA units in reserved status per SKU, split by reason (customer order, FC processing, staging), as they stand now and updated in near real-time ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). Reserved units are tied to a customer order or set aside at a fulfilment centre, and are not counted in on-hand or unsellable quantities ([Reserved Inventory report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201723860)).

## Where to find it in Seller Central

**Supply chain > Supply chain reports** (the Amazon Fulfilment Reports page; [Amazon Fulfilment reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201104750)), then **Inventory > Reserved Inventory** in the left-hand list (`/reportcentral/ReserveBreakdown/1`; **Show more...** expands the Inventory list). Classic: **Reports > Fulfilment by Amazon**. Checked in Seller Central UK on 28 Oct 2026. The UK [help page](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201723860) downloads it from the **Download** tab with **Request .csv download** or **Request .txt download**.

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_RESERVED_INVENTORY_DATA` and the marketplace ids of every store the account sells in (from Sellers `getMarketplaceParticipations`, where `isParticipating` is true and `domainName` starts `www.amazon.`, which leaves out the Non-Amazon entries), with no `dataStartTime` or `dataEndTime` and no report options; then poll `getReport` until it's done and fetch the file with `getReportDocument`, unzipping it if Amazon says GZIP. Amazon's [Request a report](https://developer-docs.amazon/sp-api/docs/request-a-report) page says the file covers every store sent unless the report type says otherwise; the [FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba) page says nothing otherwise for this one.
- **Role:** Amazon Fulfillment or Pricing for the report, either one is enough ([Role mappings for types](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)). The store list from `getMarketplaceParticipations` needs Product Listing or Selling Partner Insights in the EU ([role mappings](https://developer-docs.amazon/sp-api/docs/role-mappings)); without either, send the marketplace ids yourself.
- **How far back Amazon goes:** not stated in Amazon's docs. The report shows reserved stock as it is now, with no date range to ask for ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). Amazon can't make one for a past date, so history starts with the first copy you save. A copy your app requested in the last 90 days can still be fetched (`getReports`, then `getReportDocument`).

## Columns

Checked against a real UK seller's file in Oct 2026: 9 columns, in this order. Tab-separated text, header row first. Note the mix of underscores and hyphens in the names.

| Column | Meaning | Format and traps |
|---|---|---|
| `sku` | Your SKU. | `£` comes as `&pound;`. |
| `fnsku` | Amazon's identifier for the unit stored in and fulfilled from its fulfilment centres. | |
| `asin` | Amazon's 10-character product identifier. | |
| `product-name` | The product title. | |
| `reserved_qty` | Total units in reserved status. | Whole number. |
| `reserved_customerorders` | Units reserved for customer orders, including orders waiting on payment verification. | Whole number. |
| `reserved_fc-processing` | Units set aside at the fulfilment centre for extra processing, such as checking dimensions and weight, investigations or pending removal orders. | Whole number. |
| `reserved_staging` | Units Amazon holds briefly to keep fulfilment fast. No action needed. | Can be `0` on every row. |
| `program` | Not defined in Amazon's docs. | Can be blank on every row. |

Every column Amazon lists is in the file.

## Personal data

None: no buyer or business details. Some sellers put their cost price in the SKU, so treat SKUs as private. Rule: strip personal data before analysis and never copy it into anything shared.

## Questions it answers

- Which SKUs have units stuck in FC processing (measurement checks, investigations, pending removals)?
- How many units are tied to customer orders that haven't shipped yet?
- Which SKUs have all their units reserved, so the listing may drop out of search for a while? This report has no available count: join `fnsku` to `inventory_all` and look for `afn-fulfillable-quantity` `0` with `reserved_qty` above 0. FNSKUs with no match there can be fully reserved too: look them up in `inventory_planning` (`available` `0`).
- How did reserved stock per SKU change week on week, comparing saved snapshots?

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **No country column;** rows aren't split by store.
- **Not every FNSKU is in `inventory_all`.** FNSKUs holding only reserved units can be missing from `inventory_all`; `inventory_planning` has them.
- `£` in SKUs comes as `&pound;`.
- Lines end in CRLF, but the last line of the file has no line ending at all, so `wc -l` counts one row short and a shell `while read` loop skips that row. The text is UTF-8.

Sources: [FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba) (column list), [Reserved Inventory report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201723860) (meanings), [US Reserved Inventory report](https://sellercentral.amazon.com/help/hub/reference/external/G201723860) (`Store`).
