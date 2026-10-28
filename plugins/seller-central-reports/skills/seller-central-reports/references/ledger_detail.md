# Inventory Ledger, detailed view (`ledger_detail`)

Every FBA stock movement, one row per event: customer shipments, receipts, customer returns, removals, transfers and adjustments (lost, found, damaged, disposed).

## Where to find it in Seller Central

Checked in Seller Central UK on 28 Oct 2026: **Supply chain > Supply chain reports**, which opens Amazon Fulfilment Reports, then **Inventory Ledger** under Inventory in the left-hand list. In classic Seller Central it is **Reports > Fulfilment by Amazon**, then the same list. The ledger has **View Online** (`sellercentral.amazon.co.uk/reportcentral/LEDGER_REPORT/0`) and **Download** (`…/LEDGER_REPORT/1`) tabs ([Inventory Ledger report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G4FKT5KQWFFJ7LDN)).

On Download, choose **Detailed view** and an event type (or any). Choose a date range: the last day, 3, 7, 14, 30, 90, 180 or 365 days, or exact dates (the boxes take DD/MM/YYYY). You can filter by ASIN, FNSKU or Merchant SKU (blank means all). Then request .csv or .txt. Reports that apps asked for through the API show in the same list. A banner on the page says the ledger replaced six reports: Daily Inventory History, Monthly Inventory History, Inventory Event Detail, Inventory Adjustments, Inventory Reconciliation and Received Inventory.

Amazon calls the ledger "like a bank statement for your inventory" and says the detailed view covers movements to and from fulfilment centres, updated in near real time ([Inventory Ledger report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G4FKT5KQWFFJ7LDN)). Amazon's help says the detailed view replaced the old Inventory Adjustments, Inventory Event Detail and Received Inventory reports ([Inventory Ledger report: How to find the information you need](https://sellercentral.amazon.co.uk/help/hub/reference/external/GCRWH2T93NYDHBXU)).

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_LEDGER_DETAIL_VIEW_DATA`, `marketplaceIds` set to every store the account takes part in (for a UK account, its stores in Amazon's EU region), and `dataStartTime`/`dataEndTime` set to the window; then poll `getReport` until it's done and fetch the file with `getReportDocument` (unzip it if Amazon says GZIP). No report options needed: Amazon also accepts `eventType`, `FNSKU`, `MSKU` and `ASIN` filters, and with none, every event comes back ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)).
- **Role:** [Amazon Fulfillment](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types).
- **How far back Amazon goes:** 18 months ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba); [Inventory Ledger report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G4FKT5KQWFFJ7LDN)).

## Columns

Tab-separated text, header row first, 15 columns in this order.

**Every header and every value is wrapped in double quotes** (`"Date"`, `"MM/DD/YYYY"`), even though the file is tab-separated. Blank cells are `""`, except a blank `Reason`, which is empty with no quotes. **Quotes inside a value are not doubled:** some `MSKU` and `Title` cells hold a bare `"`. So split each line on tab (every line has exactly 15 cells) and take off only the outer pair of quotes. Never read it with a CSV reader that handles quotes: Ruby's `CSV` stops with an error, and Python's `csv` silently changes those values.

| Column | Meaning | Format and traps |
|---|---|---|
| `Date` | Date of the event | `MM/DD/YYYY`, in UK local time. |
| `FNSKU` | Amazon's fulfilment network SKU | Amazon advises reconciling on this field. |
| `ASIN` | Amazon product identifier | |
| `MSKU` | Seller's SKU | `amzn.gr.…` SKUs are Amazon's Grade and Resell units. Special characters come through as plain text (a `£` is `£`). |
| `Title` | Product title | |
| `Event Type` | Type of event | Amazon's event types: `Shipments`, `WhseTransfers`, `Adjustments`, `Receipts`, `CustomerReturns`, `VendorReturns` ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). `VendorReturns` are removals. |
| `Reference ID` | Transaction ID, such as a shipment ID or adjustment ID | For receipts it is usually the FBA shipment ID (`FBA…`, 12 characters), but some receipts carry a 15-digit number instead. For adjustments, the transaction item ID (14 digits). Blank (`""`) on every `Shipments`, `WhseTransfers`, `CustomerReturns` and `VendorReturns` row. |
| `Quantity` | Unit quantity for the transaction | Signed: units leaving stock are negative. `Shipments` and `VendorReturns` are negative, `CustomerReturns` positive. `WhseTransfers` and `Receipts` go both ways (some receipts are negative). An adjustment's sign follows its `Reason`. |
| `Fulfillment Center` | Fulfilment centre where the inventory is stored | Amazon's fulfilment centre code. |
| `Disposition` | Status of the product | Disposition codes such as `SELLABLE`, `CUSTOMER_DAMAGED` and `WAREHOUSE_DAMAGED`. |
| `Reason` | Adjustment reason | Codes, blank except on adjustments; a blank cell is empty, not `""`. E.g. `M`/`5` misplaced, `F` found, `N` found after Amazon reimbursed it, `E` damaged at Amazon FC, `D` disposed, `G` donated, `O` correction, `Q`/`P` disposition change, `4`/`3` product redefinition. Full table on the help page. Signs: `M`, `5`, `E`, `D`, `G`, `O`, `Q` and `4` are always negative; `F`, `N`, `P` and `3` always positive. |
| `Country` | Country where the inventory is stored | Two-letter code, e.g. `GB`. Stock held in EU centres can be missing (see the traps). |
| `Reconciled Quantity` | Units reconciled with other adjustment events | Blank except on misplaced, found and FC-damaged adjustments. |
| `Unreconciled Quantity` | Units not yet reconciled (found or reimbursed) | Filled on the same three reasons. A non-zero value can mark units still open, which may be worth a claim. A `0` on a misplaced (`M`, `5`) or FC-damaged (`E`) row doesn't prove the units were settled, so check them against [`reimbursements`](reimbursements.md) too. |
| `Date and Time` | Date of the event | Always 00:00 UTC, written in UK local time: `T01:00:00+0100` in summer, `T00:00:00+0000` in winter, no colon in the offset. It's a date, not the time of the event. |

Documented but not in the file: `Store`.

## Downloaded from Seller Central

**The same 15 columns, but comma-separated.** Seen in a real UK seller's Seller Central download (Oct 2026), detailed view, one month, as .csv:

- The same 15 columns, with the same names in the same order as the API file.
- **Comma-separated**, not tab-separated. No byte order mark. Lines end in CRLF, and the last line has no line ending, as in the API file.
- **Every value is in double quotes** except a blank `Reason`, which is empty with no quotes, as in the API file.
- `Date` is MM/DD/YYYY, and `Date and Time` is ISO 8601 with a `+0100` offset in summer, as in Columns.
- Special characters come through as plain text (a `£` is `£`).
- **Quotes inside values: not known.** A standard CSV reader read every row as 15 cells. But no value in that month had a `"` inside it, so it isn't known whether the CSV doubles such quotes (as a CSV should) or leaves them bare (as the API file does). Bare quotes would throw a CSV reader off, sometimes with no error, and splitting on commas breaks on titles that hold commas. Check that every row has 15 cells, and that rows whose `MSKU` or `Title` holds a `"` read correctly; if not, use the API file.

The .txt option wasn't checked. Reports requested through the API show in the same list as .txt files. For traps shared by all Seller Central downloads, such as leading zeros lost in a spreadsheet, see [seller-central-downloads](seller-central-downloads.md).

## Personal data

None of the columns hold buyer or business personal details. Some sellers put their cost price in the SKU, so treat SKUs as private.

## Questions it answers

- Which units did Amazon lose (reasons `M`, `5`) or damage (`E`) in a fulfilment centre, and which were neither found (`F`, or `N`) nor reimbursed (a line in [`reimbursements`](reimbursements.md))? Those are possible claims. An `N` row isn't a reimbursement: it often follows a reimbursement for the same FNSKU and is followed, usually the same day, by a `Reimbursement_Reversal` for it, so Amazon took the payment back. Match by FNSKU against `reimbursements` yourself. A non-zero `Unreconciled Quantity` on a lost or damaged row can mark units still open, but a `0` doesn't prove they were settled, so don't rely on that column alone.
- How many units of each FBA shipment were received, and when? (`Receipts` rows by `Reference ID` = shipment ID.) This report doesn't hold the quantity sent, so whether every unit arrived needs the shipment in Seller Central.
- What happened to one FNSKU over time: sold, returned, transferred, removed, disposed?
- Which customer returns went back to sellable stock and which came back unsellable?
- Which misplaced units were later found, and which were reimbursed instead (match against `reimbursements` by FNSKU)?

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Throttled when requested quickly.** After about ten quick `createReport` requests for this report type, Amazon refused more with HTTP 429 for about an hour, while still accepting other report types. Amazon doesn't document this. A later attempt fetched the rest.
- **Quoted values, with bare quotes inside.** Every header and value is in double quotes, but quotes inside a value aren't doubled (in some `MSKU` and `Title` cells). Split on tab and take off only the outer pair; see Columns.
- **Stock held in EU centres can be missing.** FNSKUs that [`inventory_by_country`](inventory_by_country.md) shows in EU countries, and that [`storage_fees`](storage_fees.md) bills outside `GB`, can have few or no rows here. Don't use this report for stock moving in or out of EU centres.
- Lines end in CRLF, but the last line of each file has no line ending at all, so `wc -l` counts one row short and a shell `while read` loop skips that row. The text is UTF-8.

Seen in Seller Central downloads (Oct 2026):

- **The .csv needs a different reader from the API file.** It is comma-separated with every value quoted, so the API file's tab split won't work on it. Read it with a CSV reader, then check the row lengths and any value holding a `"` (see Downloaded from Seller Central).

Sources: [FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba), [Inventory Ledger report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G4FKT5KQWFFJ7LDN), [Inventory Ledger report: How to find the information you need](https://sellercentral.amazon.co.uk/help/hub/reference/external/GCRWH2T93NYDHBXU).
