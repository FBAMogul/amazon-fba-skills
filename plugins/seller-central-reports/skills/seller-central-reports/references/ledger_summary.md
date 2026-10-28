# Inventory Ledger, summary view (`ledger_summary`)

FBA stock balances per FNSKU, disposition and country for each month: starting balance, the month's movements by type, and ending balance.

## Where to find it in Seller Central

Checked in Seller Central UK on 28 Oct 2026: the same page as [`ledger_detail`](ledger_detail.md). Go to **Supply chain > Supply chain reports**, then **Inventory Ledger** under Inventory in the left-hand list (classic Seller Central: **Reports > Fulfilment by Amazon**). On the **Download** tab (`sellercentral.amazon.co.uk/reportcentral/LEDGER_REPORT/1`), choose **Summary view**. Aggregate by location = Country (the other choice is fulfilment centre), and by time period = Monthly (Daily is among the others). Pick the dates and request .csv or .txt. The view and its settings are described on the [Inventory Ledger report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G4FKT5KQWFFJ7LDN) page, which links the report at `sellercentral.amazon.co.uk/reportcentral/LEDGER_REPORT/0` (View Online).

Amazon says the summary view shows "starting inventory balance, received inventory, customer orders, customer returns, adjustments, removals and ending balance" ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). It replaced the old Monthly Inventory History and Inventory Reconciliation reports ([Inventory Ledger report: How to find the information you need](https://sellercentral.amazon.co.uk/help/hub/reference/external/GCRWH2T93NYDHBXU)).

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_LEDGER_SUMMARY_VIEW_DATA`, `marketplaceIds` set to every store the account takes part in (for a UK account, its stores in Amazon's EU region), `dataStartTime`/`dataEndTime` set to the month, and `reportOptions`; then poll `getReport` until it's done and fetch the file with `getReportDocument` (unzip it if Amazon says GZIP).
- **Report options:** `aggregateByLocation=COUNTRY`, `aggregatedByTimePeriod=MONTHLY`. Both are Amazon's defaults. With `MONTHLY`, Amazon says `dataStartTime` must be the first of the month and `dataEndTime` the last day of the month, or the report can come back empty ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). **In a month that ends in UK summer time, end the window at 22:59:59 UTC on the last day, not 23:59:59 UTC:** 23:59:59 UTC is already the 1st of the next month in the UK, and Amazon then sends the next month's rows too (see the traps). Amazon says last month's monthly data is generally ready by the end of the 2nd ([Inventory Ledger report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G4FKT5KQWFFJ7LDN)), so a month fetched on the 1st may be incomplete; fetch it again later.
- **Role:** [Amazon Fulfillment](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types).
- **How far back Amazon goes:** the SP-API page doesn't state a limit for the summary view. The UK help page says the Inventory Ledger report holds up to 18 months ([Inventory Ledger report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G4FKT5KQWFFJ7LDN)). Another says detailed transactions are kept for the trailing 18 months and "Inventory levels in aggregate are maintained indefinitely" ([Amazon Fulfilment reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201104750)).

## Columns

Tab-separated text, header row first, 21 columns in this order.

**Every header and every value is wrapped in double quotes**, as in [`ledger_detail`](ledger_detail.md). **Quotes inside a value are not doubled:** some `MSKU` and `Title` cells hold a bare `"`, and a SKU that ends in one is written `"…""`. So split each line on tab (every line has exactly 21 cells) and take off only the outer pair of quotes. Never use a CSV reader that handles quotes: Python's `csv` reads `""` as an escaped quote, joins such a SKU to its title and shifts the rest of the row one column left without a warning; Ruby's `CSV` stops with an error.

| Column | Meaning | Format and traps |
|---|---|---|
| `Date` | The month the row covers | `MM/YYYY`, not a request date as the help page says. **A file can also hold the next month** (see the traps): keep only rows whose `Date` is the month you asked for. |
| `FNSKU` | Amazon's fulfilment network SKU | |
| `ASIN` | Amazon product identifier | |
| `MSKU` | Seller's SKU | Special characters come through as plain text (a `£` is `£`). |
| `Title` | Product title | |
| `Disposition` | Status of the product | One row per disposition; add them all for an FNSKU's total. Disposition codes such as `SELLABLE`, `CUSTOMER_DAMAGED` and `WAREHOUSE_DAMAGED`. |
| `Starting Warehouse Balance` | Units in fulfilment centres at the start of the period | Excludes units in transit between FCs. |
| `In Transit Between Warehouses` | Units in transit between fulfilment centres | Not part of the balance sum below. |
| `Receipts` | Units received in shipments to fulfilment centres | Positive, but can be negative. |
| `Customer Shipments` | Completed deliveries of customer orders Amazon fulfilled | Negative. |
| `Customer Returns` | Units buyers returned that went back into inventory | Positive. |
| `Vendor Returns` | Not defined on the help page; by name and sign, units removed by removal orders | Negative when filled. |
| `Warehouse Transfer In/Out` | Net units transshipped into and out of fulfilment centres | Signed. |
| `Found` | Missing units found and returned to inventory | Positive. |
| `Lost` | Units missing in fulfilment centres, possibly eligible for or already reimbursed | Negative. |
| `Damaged` | Not defined on the help page. Not warehouse damage. | Negative. Summed by month, it matched `ledger_detail` adjustments with reason `O` (correction) in every month checked, and reason `E` (damaged at an Amazon centre) almost never. For warehouse damage use `ledger_detail` reason `E`. |
| `Disposed` | Units removed from inventory and disposed of | Negative. |
| `Other Events` | Units identified as damaged and other dispositions not covered elsewhere | Signed. |
| `Ending Warehouse Balance` | Units in fulfilment centres at the end of the period | Equals `Starting Warehouse Balance` plus every movement column from `Receipts` to `Other Events` plus `Unknown Events`. |
| `Unknown Events` | Units whose event type isn't known, mainly events spanning two reporting periods | Signed; rare. |
| `Location` | Country, per `aggregateByLocation` | Two-letter code, e.g. `GB`. Stock held in EU centres can be missing (see the traps). |

Documented but not in the file: `Store`.

## Downloaded from Seller Central

**Not checked for the summary view.** Only the detailed view was downloaded from Seller Central (Oct 2026). Its .csv kept the API file's 15 columns, but was comma-separated, with every value quoted, no byte order mark and CRLF line endings (see [`ledger_detail`](ledger_detail.md)). The summary .csv may differ from the API file in the same way, so check it before use. This view's SKUs that end in a `"` (see Columns) are where a comma-separated, fully quoted file is most likely to go wrong. Check that every row has 21 cells, and that rows whose `MSKU` or `Title` holds a `"` read correctly. The .txt option wasn't checked either. For traps shared by all Seller Central downloads, see [seller-central-downloads](seller-central-downloads.md).

## Personal data

None of the columns hold buyer or business personal details. Some sellers put their cost price in the SKU, so treat SKUs as private.

## Questions it answers

For every question, first keep only the rows whose `Date` is the month you asked for; otherwise a file can count two months.

- How many units of each FNSKU were held in UK fulfilment centres at each month end (for stock valuation with the seller's own costs)? `Ending Warehouse Balance` leaves out units moving between centres: add `In Transit Between Warehouses`. Those units usually left in that month's `Warehouse Transfer In/Out` and came back in the next month's. Not stock held in the EU: it's missing here (see the traps).
- What moved in a month: receipts against customer shipments, returns, removals and disposals?
- Which FNSKUs show lost units (`Lost`, a negative number) in a month, to check against [`reimbursements`](reimbursements.md)? For units damaged in Amazon's centres use `ledger_detail` reason `E`: `Damaged` here isn't that.
- Does a month's opening balance equal the previous month's closing balance, and where not, what do `Unknown Events` and in-transit units explain?

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Some files held two months.** Files for months that end in UK summer time also held the next month's rows, an exact copy of that month's own file. Summing such a file's balances adds two months' stock. The cause, confirmed by a live request in October 2026: the window ended at 23:59:59 UTC, which is 00:59:59 on the 1st of the next month in the UK, so Amazon sent that month too, in full. The same month requested with the window ending at 22:59:59 UTC came back with only its own rows. Always filter `Date` to the month you asked for.
- **Throttled when requested quickly.** After about ten quick `createReport` requests for this report type, Amazon refused more with HTTP 429 for close to an hour, while accepting other report types. Amazon doesn't document this. A later attempt fetched the rest.
- **Quoted values, with bare quotes inside.** Some rows have a SKU ending in `"`, written `"…""`; Python's `csv` shifts those rows (see Columns), and when the row holds a non-zero balance, the shifted row loses real units.
- **Balances add up.** Start plus movements equalled end on every row checked, so the month's movements are complete.
- **Stock held in EU centres can be missing.** FNSKUs that [`inventory_by_country`](inventory_by_country.md) shows in EU countries, and that [`storage_fees`](storage_fees.md) bills outside `GB`, can have no row here under any location. A country's rows that do appear can be for other FNSKUs than those two reports show there, so don't match them up by country. Month-end counts per country can leave out EU-held units. For those, `inventory_by_country` gives units as they stand when you fetch it, and `storage_fees` each month's daily average per country; neither gives a month-end count.
- Lines end in CRLF, but the last line of each file has no line ending at all, so `wc -l` counts one row short and a shell `while read` loop skips that row. The text is UTF-8.

Sources: [FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba), [Inventory Ledger report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G4FKT5KQWFFJ7LDN), [Inventory Ledger report: How to find the information you need](https://sellercentral.amazon.co.uk/help/hub/reference/external/GCRWH2T93NYDHBXU), [Amazon Fulfilment reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201104750).
