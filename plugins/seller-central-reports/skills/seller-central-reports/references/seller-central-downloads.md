# Downloading reports from Seller Central

For a seller who hasn't got API access yet: where each report is in Seller Central, what the downloaded file looks like, and how it differs from the file the API sends. The report pages in this pack describe the API files; this page says where a download is different. Everything here was checked in Seller Central UK on 28 Oct 2026, in new Seller Central, with classic Seller Central paths where they were checked. Amazon moves menus often, so a path can change.

Save each file exactly as downloaded and note which report and period it covers. Keep that original unchanged and private. Some downloads hold buyer personal data, and Amazon's Data Protection Policy keeps that no longer than 30 days after order delivery unless a law requires it (see the ground rules in [SKILL.md](../SKILL.md)): after that, keep only the cleaned copy and delete the original. For those, Claude first runs a script on your computer that copies the file without the buyer columns (found by name in the header row, not by position), then reads rows only from the copy and never prints the original. The downloads with buyer columns:

- [All Orders](orders.md): `ship-city`, `ship-state`, `ship-postal-code`.
- [Date Range transaction report](date_range_transactions.md): `order city`, `order state`, `order postal`.
- [Amazon VAT Transactions Report](vat_transactions.md): `BUYER_NAME`, `ARRIVAL_ADDRESS`, `ARRIVAL_CITY`, `ARRIVAL_POST_CODE`, `BUYER_VAT_NUMBER`, and `DEPATURE_CITY` and `DEPARTURE_POST_CODE` (the buyer's location on return rows).
- [Amazon Fulfilled Shipments](fba_shipments.md): `buyer-email`, and `ship-city`, `ship-state`, `ship-postal-code`.

Any other report's page names its own in its Personal data section.

## Where the reports are

In new Seller Central most reports sit at the end of a ☰ menu section: **Products > Product reports**, **Supply chain > Supply chain reports** (the Amazon Fulfilment Reports page, which lists the FBA reports, the VAT report and more), **Orders > Order reports** and **Finance > Finance reports** (Payment reports and the Tax document library). **Business Reports** and **Custom analytics** are not in the ☰ menu: open **Site map** (at the bottom of the ☰ menu), then **Other resources**. The home page also has an **All reports** drop-down at the top right.

Classic Seller Central (the link at the bottom of the ☰ menu) has a **Reports** menu (Business Reports, Custom analytics, Fulfilment by Amazon, Advertising Reports, Return Reports, Custom Reports, Inventory Reports, Tax Document Library, Manage Taxes, Manage your VAT, Manage your EPR Compliance, Selling economics and fees, SKU Economics report) and a **Payments** menu (Payments, Reports Repository and more). Its banner says switching back to classic as your default will soon no longer be possible.

On the Amazon Fulfilment Reports page each report has a download page, and some also have a "View online" page. All Orders is listed there under Most Popular Reports, but it lives under Orders > Order reports.

## Traps in every download

- **CSV or TXT.** Many report pages offer both. Amazon's own note on the fulfilment report pages says a .csv opens in a spreadsheet, but the spreadsheet drops the leading zeros from SKUs, ASINs and FNSKUs, and to download .txt if you need them. So never open a report in a spreadsheet and save it before giving it to Claude: a SKU that lost its zeros no longer joins to anything.
- **Byte order marks.** Many files start with an invisible UTF-8 byte order mark: Transaction View, Business Reports, the FBA Inventory report, All Listings, SKU Economics, the Date Range report, the VAT report and Voice of the Customer. The seller fee invoice CSV, the Inventory Ledger CSV, All Orders and the settlement Flat File V2 had none. Strip it, or the first column name doesn't match.
- **Date order differs by page and by file.** A date like `03/04/2026` can't be read safely until you know which file it came from.
  - Form date boxes in **DD/MM/YYYY:** Business Reports, Transaction View, Inventory Ledger, All Statements.
  - Form date boxes in **MM/DD/YYYY** (US order): Reports Repository, All Orders, Performance Notifications.
  - Inside files, **DD/MM/YYYY:** Transaction View, Business Reports.
  - Inside files, **MM/DD/YYYY:** Inventory Ledger `Date`, seller fee invoice CSV, SKU Economics, and the dates in Custom analytics column headers.
  - Inside files, other styles: VAT report `DD-MM-YYYY` with dashes; Date Range report `1 Sept 2026 09:30:00 UTC`; Voice of the Customer `Last updated` as `1 Oct 2026`; Inventory Ledger `Date and Time` and All Orders `purchase-date` ISO 8601 with an offset.
- **Business Reports numbers are text.** Money is written as `£1,234.56`, counts of 1,000 or more as `"1,234"` with a thousands comma inside quotes, and percentages as `12.34%`. Take out the £, the commas and the % before any maths. The Date Range report also puts thousands commas in quoted amounts.
- **The Date Range report has nine lines of preamble** before its header row ([date_range_transactions](date_range_transactions.md)).
- **File names rarely give the period.** Business Reports (`BusinessReport-DD-MM-YYYY.csv`) and All Listings (`All+Listings+Report_MM-DD-YYYY.txt`, US date order) are named by the download date. All Orders and the settlement Flat File V2 are a bare number, SKU Economics a long id, and Voice of the Customer always `Listing_Summary_Report.csv`. Only Transaction View and the Date Range report name the period. Rename each file with its report and period as soon as it's downloaded.
- **Some reports take a while.** All Listings and All Orders say they can take up to 45 minutes. The Date Range, VAT, SKU Economics and Inventory Ledger reports took about a minute. Voice of the Customer's first click only starts the report; click again later to download it.
- **Check what the All Orders file covers.** When an exact range was typed on the All Orders form, the end date reset and the report covered one day only. The request table's **Date Range Covered** column shows the real range (in British Summer Time; the requested and completed times are GMT): check it before trusting the file. Each All Orders download covers at most 30 days.
- **Reports that apps requested through the API appear too.** Seller Central's report lists also show reports apps asked for (for example on the All Listings, Amazon-fulfilled Inventory and Inventory Ledger pages). A seller with a connected app sees requests they didn't make; pick out your own by its date and range.
- **Download links expire.** SKU Economics links work for 30 days. Old Reports Repository requests show Expired with a "Request Again" link.
- **Old report names.** Amazon's notes say the Inventory Ledger replaced six reports (Daily Inventory History, Monthly Inventory History, Inventory Event Detail, Inventory Adjustments, Inventory Reconciliation, Received Inventory), and the FBA Inventory report replaced the Inventory Health, Inventory Age and Excess Inventory reports. An older guide that names one of those means one of these.
- **Business Reports is being replaced.** A banner on Business Reports points sellers to Custom Analytics and says to switch by year-end.
- **No download on some pages.** Orders with Defects and Performance Notifications showed no download button when checked.

## Downloads checked

Seen in a real UK seller's Seller Central downloads (Oct 2026). "Not recorded" means the file name wasn't noted.

| Report | Where (new Seller Central; classic) | File name | Format | Compared with the API file | Page |
|---|---|---|---|---|---|
| Business Reports: Sales and traffic (by date) | Site map > Other resources > **Business reports** (`/business-reports`), By Date > Sales and traffic. Classic: Reports > Business Reports. | `BusinessReport-DD-MM-YYYY.csv` (download date) | CSV, byte order mark, quotes only where needed, no line end after the last row. 18 columns. | The API sends JSON with different names, money as objects and percentages as numbers. This CSV writes them as text and has fewer metrics (see below). | [sales_and_traffic](sales_and_traffic.md) |
| Business Reports: Detail page sales and traffic by child item | Same page, By ASIN. | As above | CSV, byte order mark. 21 columns, no SKU column. | As above. | [sales_and_traffic](sales_and_traffic.md) |
| Transaction View | Finance > Finance reports > Payment reports, **Transaction View** tab (`/payments/event/view`). | `Transactions for time period DD_MM_YYYY - DD_MM_YYYY.csv` (the period) | CSV, byte order mark, every field quoted. 10 columns. At most 600 transactions per download. | Totals per transaction only; `listTransactions` gives every breakdown. | [transactions](transactions.md); columns below |
| Date Range Transaction report | Finance > Finance reports > Payment reports, **Reports Repository (New)** tab, Report Type Transaction. Classic: Payments > Reports Repository. | `YYYYMonD-YYYYMonDCustomTransaction.csv` (the period) | CSV, byte order mark, nine preamble lines. 29 columns. | No report type; the API route to the same transactions is `listTransactions`. | [date_range_transactions](date_range_transactions.md) |
| Settlement report (Flat File V2) | Finance > Finance reports > Payment reports, **All Statements** tab (`/payments/past-settlements`), **Download Flat File V2** (its drop-down also offers XML and the old flat file). | `<number>.txt` | Tab-separated, no byte order mark, LF. 24 columns. | The same 24 columns as `GET_V2_SETTLEMENT_REPORT_DATA_FLAT_FILE_V2`. | [settlements](settlements.md) |
| Seller fee invoice CSV | Finance > Finance reports, **Tax document library** tab, Seller Fee Invoices, rows marked CSV. Classic: Reports > Tax Document Library. | Not recorded | CSV, no byte order mark, every field quoted. 7 columns. | No API route found for seller fee invoices ([seller-central-map](seller-central-map.md)); the fees themselves are in `transactions`. | Columns below |
| Amazon VAT Transactions Report | Supply chain > Supply chain reports, **Tax** (or Favourite) > Amazon VAT Transactions Report (`/reportcentral/VAT_TRANSACTION/1`). Classic: Reports > Fulfilment by Amazon. | Not recorded | CSV (.txt offered), byte order mark, LF. 95 columns. | `GET_VAT_TRANSACTION_DATA`, which needs the restricted Tax Invoicing role. | [vat_transactions](vat_transactions.md) |
| Inventory Ledger, Detailed view | Supply chain > Supply chain reports, Inventory > **Inventory Ledger** (`/reportcentral/LEDGER_REPORT/1`), Download, Detailed view. Classic: Reports > Fulfilment by Amazon. | Not recorded | CSV (.txt offered), no byte order mark, CRLF, no line end after the last row. | The same 15 columns and names as `GET_LEDGER_DETAIL_VIEW_DATA`, but comma-separated, not tab (see below). | [ledger_detail](ledger_detail.md) |
| FBA Inventory report | Supply chain > Manage Amazon inventory > Reports > **Inventory report**, which opens Supply chain reports > Inventory > FBA Inventory (`MANAGE_INVENTORY_HEALTH`). Classic: Inventory > FBA Inventory (the same page). | Not recorded | CSV, byte order mark, every value quoted. | The same 89 columns, names and order as `GET_FBA_INVENTORY_PLANNING_DATA`, but comma-separated, not tab. | [inventory_planning](inventory_planning.md) |
| All Listings Report | Products > **Product reports** (`/listing/reports`), "Request an Inventory report" > All Listings Report. Classic: Reports > Inventory Reports. | `All+Listings+Report_MM-DD-YYYY.txt` (download date) | Tab-separated, byte order mark, LF. 31 columns. No CSV offered. | Identical to `GET_MERCHANT_LISTINGS_ALL_DATA`. | [listings_all](listings_all.md) |
| All Orders | Orders > **Order reports**, All orders tab (`/order-reports-and-feeds/reports/allOrders`). Classic: Orders > Order Reports. | `<number>.txt` | Tab-separated, no byte order mark, CRLF. 35 columns. | Adds `url` between `order-channel` and `ship-service-level`, and the last header is `is-prime ` with a trailing space. The pack's API column list has neither; the API file wasn't checked again. | [orders](orders.md) |
| SKU Economics report | Finance > **Profit analytics**, Selling Economics, SKU Economics report tab (`/sereport`). Classic: Reports > SKU Economics report. | `<long id>.amzn1.tortuga.4.eu.csv` | CSV, byte order mark, LF. 69 columns with every data box ticked. | The API route is Data Kiosk JSON, with completely different names. | [seller_economics](seller_economics.md); columns below |
| Custom analytics export | Site map > Other resources > **Custom analytics**, a dashboard widget's Export. Classic: Reports > Custom analytics. | `kpi-chart-export-today-<date>.xlsx` | .xlsx workbook, not CSV. 21 columns. | No API route in this pack. | Columns below |
| Voice of the Customer | Products > Analyse reviews, **Voice of Customer** tab, Download data (also in Account health). Classic: Performance > Voice of the Customer. | `Listing_Summary_Report.csv` (always) | CSV, byte order mark, CRLF. 14 columns. | No API route found ([not-in-the-api](not-in-the-api.md)). | Columns below |

### Notes on reports that have their own page

- **All Orders form.** Choose Order date or Last Updated Date, then a range (last day, 3, 7, 14 or 30 days, or exact dates). Amazon says the report covers the past two years, with older orders in the Archived Orders report, and the UI doesn't offer XML. An **Add or remove order report columns** link can change the download's columns. Read columns by trimmed name, never by position.
- **Inventory Ledger CSV.** Every value is quoted except a blank `Reason` (empty, no quotes); `Date` is MM/DD/YYYY and `Date and Time` ISO 8601 with a `+0100` offset, as in the API file; a `£` in a SKU is a plain `£`. A standard CSV reader read every row of the file checked as 15 cells. But that month had no quote marks inside values, so whether the CSV doubles them (the API file doesn't) is unknown: check every row has 15 cells. The form offers a Summary view (by country or fulfilment centre, daily and longer periods) or a Detailed view, over the last day up to 365 days, or exact dates.
- **FBA Inventory report and All Listings.** A `£` in a SKU is a plain `£`.
- **Settlement Flat File V2.** All Statements lists each settlement period (about two weeks) with its balances and payout; a period still open says it is available after the settlement closes.

### Business Reports CSV columns

The download seems to follow the columns chosen under Show/Hide Columns, so a file can have more or fewer than these.

- **Sales and traffic (by date), 18 columns:** `Date`, `Ordered Product Sales`, `Ordered product sales – B2B`, `Units ordered`, `Units ordered – B2B`, `Total order items`, `Total order items – B2B`, `Average sales per order item`, `Average sales per order item – B2B`, `Average units per order item`, `Average units per order item – B2B`, `Average selling price`, `Average selling price – B2B`, `Sessions – Total`, `Sessions – Total – B2B`, `Order Item Session Percentage`, `Order item session percentage – B2B`, `Average offer count`.
- **Detail page sales and traffic by child item, 21 columns:** `(Parent) ASIN`, `(Child) ASIN`, `Title`, `Sessions – Total`, `Sessions – Total – B2B`, `Session percentage – Total`, `Session percentage – Total – B2B`, `Page views – Total`, `Page views – Total – B2B`, `Page views percentage – Total`, `Page views percentage – Total – B2B`, `Featured Offer (Buy Box) percentage`, `Featured Offer (Buy Box) percentage – B2B`, `Units ordered`, `Units ordered – B2B`, `Unit Session Percentage`, `Unit session percentage – B2B`, `Ordered Product Sales`, `Ordered product sales – B2B`, `Total order items`, `Total order items – B2B`.
- **Names use an en dash (`–`), not a hyphen, and capitals vary** (`Ordered Product Sales` but `Ordered product sales – B2B`). Match names exactly.
- **Compared with the API file** ([sales_and_traffic](sales_and_traffic.md)): the same kinds of figure under plain-English names. The columns seen have no page views, refunds, claims, shipped or feedback figures in the by-date file, and no browser or app split in either file; the JSON has them all. The by-ASIN pages show a Growth opportunities column that isn't in the CSV.

### SKU Economics CSV columns

Seen with one store selected, MSKU level, one month and every data box ticked. Fewer boxes probably give fewer columns (not checked). One row per MSKU in the catalogue, including those with no sales in the period.

- **Product and sales:** `Amazon store` (a country code, e.g. `GB`), `Start date`, `End date`, `Parent ASIN`, `ASIN`, `FNSKU`, `MSKU`, `Currency code`, `Average sales price`, `Units sold`, `Units returned`, `Net units sold`, `Sales`, `Net sales`.
- **Fees, three columns each** (`<fee> per unit`, `<fee> quantity`, `<fee> total`), for: Aged inventory surcharge; Base fulfilment fee; Base monthly storage fee; Digital Services Fee (FBA Fulfilment fees); Digital Services Fee (Selling on Amazon fees); FBA Inventory Reimbursement; FBA disposal order fee; FBA removal order fee; Fuel and logistics-related surcharge; Fulfilment by Amazon fulfilment fees; Inbound Transportation Fee; Inbound Transportation Program Fee; Monthly storage fee; Referral Fee Refunds; Referral fee; Refund administration fee; Storage utilisation surcharge.
- **Costs and result:** `Cost of goods sold per unit`, `Miscellaneous cost per unit`, `Net proceeds total`, `Net proceeds per net unit sold`.
- Dates are MM/DD/YYYY. Numbers are plain, and **fees are positive numbers**, so subtract them. `FNSKU` is blank at MSKU level. Cost of goods is blank unless the seller has entered their costs (Additional financial inputs). Whether these figures include VAT wasn't checked: [seller_economics](seller_economics.md) has the VAT traps in the API version.
- The form takes one or more stores, an aggregation level (MSKU by default) and a range (last 7 or 30 days, a month, a quarter, a year or a custom range), with a tick-box for each kind of data. The generation time is shown in the browser's time zone.

## Files with no page of their own

### Transaction View

**Finance > Finance reports > Payment reports > Transaction View.** Filter by account type, transaction type, transaction status and a period (a custom range in DD/MM/YYYY, a number of past days, or a settlement period), then **Download**. It shows deferred and released status. Amazon's banner says the download holds at most 600 transactions and, for a complete list and more columns, points to All Statements or the Reports Repository ([date_range_transactions](date_range_transactions.md)). The page itself stops loading at 10,000 transactions for a period.

10 columns, every field quoted, with a byte order mark: `Date`, `Transaction status`, `Transaction type`, `Order ID`, `Product Details`, `Total product charges`, `Total promotional rebates`, `Amazon fees`, `Other`, `Total (GBP)`.

- `Date` is DD/MM/YYYY.
- `Product Details` is cut short with `...`, so it won't match a full title.
- Amounts are plain numbers with no £ sign and as few decimals as needed (`0`, `-1.5`, `12.34`), not a fixed two. The currency is only in the `Total (GBP)` header.
- No buyer details.
- The API route to the same transactions, with every fee broken down, is `listTransactions` ([transactions](transactions.md)).

### Seller fee invoice CSV

**Finance > Finance reports > Tax document library > Seller Fee Invoices** (there is also a Seller Fee Credit Notes tab). The list shows the 1,000 most recent invoices, with Load More for older ones, and for each the payer name, VAT numbers, supplier (Amazon EU S.à r.l., UK Branch), store and period. PDFs are unchanged; Amazon says a transaction-level CSV breakdown is now available on rows marked CSV. Advertising tax documents have moved to the Ads console.

7 columns, every field quoted, no byte order mark: `Transaction Date`, `Transaction ID`, `Order ID`, `Fees Invoice Number`, `Marketplace`, `Fee ID`, `Total Fees (VAT-Inclusive)`.

- `Transaction Date` is MM/DD/YYYY.
- `Order ID` is `-` on fees not tied to an order.
- `Fee ID` names the fee, e.g. `FBA Inventory Storage`.
- `Total Fees (VAT-Inclusive)` has four decimal places and, as its name says, includes VAT.
- A month's invoice can hold a row dated the last day of the month before, so don't assume every row falls inside the invoice period.
- Personal data: none about buyers. The payer name, VAT numbers and invoice numbers are the seller's business details: keep them private.

### Custom analytics export

**Site map > Other resources > Custom analytics** (classic: Reports > Custom analytics). Amazon's dashboards (Overall business performance, Sales and pricing, Conversion Analysis and more), the seller's own dashboards and AI-built dashboards; every widget has an Export, and dashboards can be shared by link within the organisation. Checked: Overall business performance > Sales Performance > Export, for today.

The file is an **.xlsx workbook, not CSV**, named `kpi-chart-export-today-<date>.xlsx`, with 21 columns.

- The first column is `Hour`. Every other column is one metric for one period, with the period and its date in the header, e.g. `Ordered units (Today so far MM/DD/YYYY)`, and the same metric for Yesterday, Same day last week and Same day last year. A comparison header can also say its figures are matched through the same hour as today's (`…; matched row through the hour starting …`). Money headers carry the currency, e.g. `Ordered revenue (…)(£)`.
- Dates in the headers are MM/DD/YYYY, so the column names change every day: find columns by metric and period, not by exact name.
- Row 2 is `Total`; the hourly rows follow. Numbers are stored as numbers.
- No API route for this export is in this pack. Its sales and traffic figures overlap Business Reports ([sales_and_traffic](sales_and_traffic.md)) and Seller Analytics ([seller_analytics](seller_analytics.md)), but neither gives hourly figures.

### Voice of the Customer

**Products > Analyse reviews > Voice of Customer** (also in Account health, and classic Performance > Voice of the Customer). The page shows CX Health for each listing. **Download data** first says the summary report is being created; click again later to download it.

`Listing_Summary_Report.csv`, with a byte order mark and CRLF line ends. 14 columns:

| Column | Format and traps |
|---|---|
| `Product name` | |
| `ASIN` | |
| `SKU` | **Holds the FNSKU** (`X00…`), not the seller SKU, on every row. Join it to FNSKU columns, never to SKUs. |
| `Condition` | Mixed case, e.g. `New` and `new`. Compare without case. |
| `Fulfilled by` | |
| `NCX rate` | Text with a % sign, e.g. `0%` or `12.34%`. |
| `Negative Customer Experience (NCX) orders` | |
| `Total orders` | |
| `Star rating` | Can be `Not available`. |
| `Return rate` | Like `NCX rate`; can be `Not available`. |
| `Top NCX reason` | The reason and its share run together with no space (the reason's words, then a percentage such as `50%`), or `--`. |
| `Last updated` | Day, month abbreviation, year, e.g. `1 Oct 2026`. |
| `Customer Experience (CX) Health` | `Excellent`, `Good`, `Fair`, `Poor` or `Very poor`. |
| `Return Badge Displayed` | `Yes`, `At risk` or `--`. |

No buyer details are in the download. No API route was found for NCX or CX Health ([not-in-the-api](not-in-the-api.md)).
