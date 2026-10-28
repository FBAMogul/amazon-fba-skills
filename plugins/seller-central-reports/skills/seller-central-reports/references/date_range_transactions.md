# Payments Date Range Transaction report (`date_range_transactions`)

Every payments line posted in a date range you choose (orders, refunds, fees, adjustments, transfers), one row per line, with its settlement, its fee columns and whether it is released or deferred, as a CSV from the Payments Reports Repository.

## Where to find it in Seller Central

**Finance > Finance reports > Payment reports**, then the **Reports Repository (New)** tab (checked in Seller Central UK on 28 Oct 2026). In classic Seller Central: **Payments > Reports Repository**.

Choose the Account Type (for example Standard orders), Report Type **Transaction** (the other two are Summary and Deferred Transaction), a Reporting Date Range (a custom range or a month) and, if you like, a Report Tag, then **Request Report**. The table below the form lists each request with its request date, reporting start and end dates, account type, report type, tag and status: In Progress until it is ready, and Expired, with a "Request Again" link, once the download has lapsed. A one-month request took about a minute.

**The custom range boxes take MM/DD/YYYY** (US order), unlike Transaction View's DD/MM/YYYY boxes on the next tab. Check the reporting start and end dates in the table before you download.

Amazon's UK help says Date Range reports cover "up to 365 days at a time, going back as far as 2012", and that the Summary report is a PDF ([Date Range reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200989190)).

## Getting it through the API

- **Call / report type:** none. The pack's [report-catalogue](report-catalogue.md) lists no report type for the date-range transaction report, and [seller-central-map](seller-central-map.md) says the date-range transaction and summary reports "have no report type; use the Finances API instead". The API route to the same transactions is Finances API 2024-06-19 `listTransactions`: see [transactions](transactions.md). It sends JSON, one object per transaction with nested breakdowns, so its fields don't map one to one onto the columns below.
- **Not the Deferred Transaction report.** The Repository's Deferred Transaction report type is a separate report, `GET_DATE_RANGE_FINANCIAL_HOLDS_DATA`. The catalogue says it "Must first be requested in Seller Central" (Payments Reports Repository) and is then fetched by API, needs the Finance and Accounting role, and covers "Up to 10 years in one report" ([report-catalogue](report-catalogue.md), from Amazon's [Payment reports](https://developer-docs.amazon/sp-api/docs/report-type-values-payment)).
- **Role:** for `listTransactions`, [Finance and Accounting](https://developer-docs.amazon/sp-api/docs/role-mappings).
- **How far back Amazon goes:** in Seller Central, up to 365 days per request, back to 2012 ([Date Range reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200989190)). Through `listTransactions`, at most 180 days per request, and Amazon doesn't state a history limit ([transactions](transactions.md)).

## Columns

Seen in a real UK seller's Seller Central downloads (Oct 2026), for a range starting after 1 Jan 2025. Comma-separated, starting with a UTF-8 byte order mark.

**Nine lines of preamble come before the header.** Each is one quoted sentence in a single cell: what the report includes (Amazon Marketplace, FBA and Amazon Webstore transactions), that amounts are in GBP unless stated, "Definitions:", Amazon's definitions of Date/Time, Sales tax collected, Selling fees, Other transaction fees and Other, and a note that reports starting on or after 1 Jan 2025 are indexed by posted date, include released and deferred transactions, and have two new columns. Read Amazon's definitions from the file itself. The header is the row whose first cell is `date/time`.

29 columns, in this order. Beyond the preamble's definitions Amazon gives no meanings; "by name" means read from the name and the values.

| Column | Meaning | Format and traps |
|---|---|---|
| `date/time` | When the line was posted (defined in the preamble). | Day, month abbreviation, year, time and `UTC`, e.g. `1 Sept 2026 09:30:00 UTC`. **September is `Sept`**, four letters, so a parser that expects three-letter months fails. |
| `settlement id` | By name, the settlement the line belongs to. | |
| `type` | By name, the kind of line. | Values include: `Order`, `Refund`, `Adjustment`, `Fulfilment by Amazon (FBA) transaction fees`, `FBA Inventory Fee`, `Transfer`, `Order_Retrocharge`, `Service Fee`. |
| `order id` | By name, the Amazon order id. | |
| `sku` | By name, the seller's SKU. | Some sellers put their cost price in the SKU, so treat SKUs as private. |
| `description` | By name, what the line is for. | |
| `quantity` | By name, units. | |
| `marketplace` | By name, the Amazon store. | `amazon.co.uk`, and on some rows `Amazon.co.uk`: compare without case. Blank on rows that aren't orders. |
| `fulfilment` | By name, who fulfilled the order. | e.g. `Amazon`; blank on rows that aren't orders. |
| `order city` | The buyer's city. | Personal: never copy. Often filled on order rows. |
| `order state` | The buyer's county or region. | Personal: never copy. |
| `order postal` | The buyer's postcode. | Personal: never copy. |
| `tax collection model` | By name, who collected the VAT. | Blank, or `MarketplaceFacilitator`. |
| `product sales` | By name, the product price charged. | Money (see below). |
| `product sales tax` | By name, the VAT on `product sales`. | |
| `postage credits` | By name, delivery charged to the buyer. | Called postage here but shipping in the next column. |
| `shipping credits tax` | By name, the VAT on delivery. | |
| `gift wrap credits` | By name, gift wrap charged to the buyer. | |
| `giftwrap credits tax` | By name, the VAT on gift wrap. | `giftwrap` is one word here, two in the column before. |
| `promotional rebates` | By name, promotion discounts. | |
| `promotional rebates tax` | By name, the VAT on promotion discounts. | |
| `marketplace withheld tax` | By name, tax Amazon kept as the marketplace. | |
| `selling fees` | Defined in the preamble. | Fees are negative. **Look to include VAT,** with no column for the fee VAT (see the traps). |
| `fba fees` | By name, FBA fees. | Negative. Look to include VAT, like `selling fees`. |
| `other transaction fees` | Defined in the preamble. | Negative. Look to include VAT, like `selling fees`. |
| `other` | Defined in the preamble. | |
| `total` | By name, the line's total. | |
| `Transaction status` | Whether the money is released. | `Released` or `Deferred`. Capital T, unlike the columns before it. With `Transaction Release Date`, probably the two new columns the preamble mentions. |
| `Transaction Release Date` | By name, when the line was released. | Same format as `date/time`, e.g. `10 Sept 2026 14:00:00 UTC`. Filled on every `Released` row, blank on `Deferred` rows. |

**Money columns** are plain numbers with no £ sign, in GBP unless the file says otherwise. Fees are negative. Values of 1,000 or more carry a thousands comma inside quotes, e.g. `"1,234.56"`: take the comma out before adding up.

## Personal data

`order city`, `order state` and `order postal` are the buyer's location, often filled on order rows. There are no buyer names or emails. Strip them before analysis and never copy them into anything shared. Some sellers put their cost price in the SKU, so treat SKUs as private.

## Questions it answers

- What did the account take in sales, pay in fees and refund over any period up to a year, in one file, without API access?
- Which fees (selling, FBA, other) did each order carry?
- Which lines are still deferred (`Transaction status` `Deferred`)? The file doesn't say when they will be released: `Transaction Release Date` is blank on them.
- Which lines belong to each settlement (`settlement id`)?
- Which account-level charges hit the account in a month (`type` `FBA Inventory Fee`, `Service Fee`)?

## Traps seen in real data

Seen in a real UK seller's Seller Central downloads (Oct 2026):

- **Preamble before the header.** Nine single-cell lines come first, so a reader that takes row 1 as the header, or expects 29 cells on every row, fails. Skip to the `date/time` row.
- **Byte order mark.** The file starts with a UTF-8 byte order mark; strip it, or the first preamble line starts with an invisible character.
- **`Sept`** in `date/time` and `Transaction Release Date` (see Columns).
- **Thousands commas** inside quoted amounts of 1,000 or more.
- **Released and deferred lines are both in.** For a range starting on or after 1 Jan 2025 the file is indexed by posted date and holds `Deferred` lines beside `Released` ones, so the sum of `total` isn't what was paid out. Filter on `Transaction status` first. How a deferred line and its later release show up across two files wasn't checked; [transactions](transactions.md) describes the pairs in the API data.
- **`Transfer` rows are bank payouts; they aren't sales or fees.** Leave them out of earnings and fee totals ([transactions](transactions.md) has the same type).
- **Fees look to include VAT, and the file doesn't split it out.** UK referral fees in `selling fees` came out at exactly the category rate plus 20% VAT on it: a 15% rate shows as 18% of the VAT-inclusive sale. No column holds the VAT on fees, so treat `selling fees`, `fba fees`, `other transaction fees` and fee amounts in `other` as including it. A seller who reclaims fee VAT (VAT-registered on standard accounting) reclaims it through Amazon's seller fee invoices (Finance > Finance reports > Tax document library > Seller Fee Invoices; the fee rows in [vat_transactions](vat_transactions.md) have it too), so their fee cost is the fee before VAT. A Flat Rate Scheme or non-registered seller can't reclaim it, so for them the fees as shown are the cost. See the VAT section of [SKILL.md](../SKILL.md).
- **Column names aren't consistent:** `postage credits` beside `shipping credits tax`, `gift wrap credits` beside `giftwrap credits tax`, and the last two columns capitalised. Find columns by their exact name.
- **`marketplace` mixes `amazon.co.uk` and `Amazon.co.uk`.**
- **The date boxes are MM/DD/YYYY.** Typed as DD/MM/YYYY, 01/09/2026 asks for 9 January, not 1 September.
- **The file name gives the period**, e.g. `2026Sep1-2026Sep30CustomTransaction.csv`, unlike many other downloads ([seller-central-downloads](seller-central-downloads.md)).

Sources: [Date Range reports (UK help)](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200989190), [Payment reports (SP-API)](https://developer-docs.amazon/sp-api/docs/report-type-values-payment), [listTransactions](https://developer-docs.amazon/sp-api/reference/listtransactions).
