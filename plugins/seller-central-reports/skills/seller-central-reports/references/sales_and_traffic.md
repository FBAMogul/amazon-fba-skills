# Business Reports: Sales and Traffic (`sales_and_traffic`)

Amazon's Sales and Traffic Business Report for one store: sales metrics (ordered product sales, units, refunds, claims) and traffic metrics (sessions, page views, Buy Box percentage), with B2B splits ([Amazon](https://developer-docs.amazon/sp-api/docs/report-type-values-analytics)), as one entry per day for the whole store and one entry per ASIN for the whole period requested.

## Where to find it in Seller Central

Checked in Seller Central UK on 28 Oct 2026: Business Reports is not in the new main menu. Open **Site map** (at the bottom of the menu), then **Other resources > Business reports**. In classic Seller Central it is **Reports > Business Reports** ([UK help](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200633570)).

The left-hand menu has the By Date reports (**Sales and traffic**, **Detail page sales and traffic**, **Seller performance**) and the By ASIN reports (**Detail page sales and traffic**, and the same by parent item and by child item) ([UK help](https://sellercentral.amazon.co.uk/help/hub/reference/external/G202142060)). Choose a view by day, week or month and a custom date range (the boxes take DD/MM/YYYY), then **Download (.csv)**. The By ASIN pages also take up to 100 ASINs in an optional search box.

A banner at the top of Business Reports asks sellers to move to **Custom analytics** (also under Site map > Other resources), which Amazon says has the Business Reports metrics and more, and to switch "by year-end". So the page may not last beyond 2026.

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_SALES_AND_TRAFFIC_REPORT`, **one** store's marketplace id per request, and `dataStartTime`/`dataEndTime` set to the period; then `getReport` until the report is done, and `getReportDocument` (unzip it if Amazon says GZIP). Amazon's schema: "This report type supports only one marketplaceId per report. Specifying multiple marketplaces will result in failure to generate the report" ([schema](https://github.com/amzn/selling-partner-api-models/blob/main/schemas/reports/sellerSalesAndTrafficReport.json)). So ask store by store.
- **Report options:** `dateGranularity` and `asinGranularity`. Amazon's defaults are `DAY` and `PARENT` ([Amazon](https://developer-docs.amazon/sp-api/docs/report-type-values-analytics)). The columns below are from requests with `dateGranularity` = `DAY` and `asinGranularity` = `CHILD`; `SKU` granularity adds a `sku` field.
- **Role:** Brand Analytics ([Amazon](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)).
- **How far back Amazon goes:** `dataStartTime` no more than two years before the request, or the request is cancelled ([Amazon](https://developer-docs.amazon/sp-api/docs/report-type-values-analytics)). Seller Central's Business Reports are "generally available for up to two years" ([UK help](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200633570)).
- **Rate limit:** Amazon accepts this report three times every five minutes ([Amazon](https://developer-docs.amazon/sp-api/docs/report-type-values-analytics)).
- **Delay:** Business Reports lag real-time data by 24 hours ([UK help](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200633570)).

## Columns

JSON, not tab-separated. Field paths as seen in a real UK seller's files (Oct 2026), one request per store per calendar month.

`salesAndTrafficByDate` holds one entry per day for the whole store. `salesAndTrafficByAsin` has no date field in Amazon's schema, so each child ASIN's entry covers the whole period requested ([schema](https://github.com/amzn/selling-partner-api-models/blob/main/schemas/reports/sellerSalesAndTrafficReport.json)).

**This is where page views live:** per store per day in `salesAndTrafficByDate[].trafficByDate`, and per child ASIN for the period in `salesAndTrafficByAsin[].trafficByAsin`. [seller_analytics_asin](seller_analytics_asin.md) has them per ASIN per store per month too ([seller_analytics](seller_analytics.md) only per store), but its browser page views and sessions are far too low, so take the browser vs app split from here.

Money fields are objects with `amount` and `currencyCode` (ISO 4217, the store's currency). Percentage fields are on a 0 to 100 scale (Buy Box up to `100.0`), not fractions. Each field ending `B2B` is the same metric for Amazon Business customers only. `[]` marks an array; its entries all share the shape listed.

| Column | Meaning | Format and traps |
|---|---|---|
| `reportSpecification.reportType` | The report type. | `GET_SALES_AND_TRAFFIC_REPORT`. |
| `reportSpecification.reportOptions.dateGranularity` | Granularity of the by-date section. | e.g. `DAY`. |
| `reportSpecification.reportOptions.asinGranularity` | Granularity of the by-ASIN section. | e.g. `CHILD`. |
| `reportSpecification.dataStartTime` | Start date of the report. | A date, `YYYY-MM-DD`. |
| `reportSpecification.dataEndTime` | End date of the report. | A date, `YYYY-MM-DD`. |
| `reportSpecification.marketplaceIds[]` | The store the report covers. | One id only, e.g. `A1F83G8C2ARO7P` (UK). |
| `salesAndTrafficByDate[].date` | The start date of the period of the aggregated data. | `YYYY-MM-DD`. One entry per day, even with no sales. **UK time** in the UK store, not UTC (see Traps), so the months are UK time too. |
| `salesAndTrafficByDate[].salesByDate.orderedProductSales`, `…B2B` | Ordered product sales. | Money. Seller Central's formula: Sum(ItemPrice * UnitsOrdered). **Includes VAT,** like `orders`' `item-price`. |
| `salesAndTrafficByDate[].salesByDate.unitsOrdered`, `…B2B` | Number of units ordered in the period. | Count. |
| `salesAndTrafficByDate[].salesByDate.totalOrderItems`, `…B2B` | Number of order items ordered in the period. | Count. An order of 2 of book A and 3 of book B is 1 order, 2 order items, 5 units. |
| `salesAndTrafficByDate[].salesByDate.averageSalesPerOrderItem`, `…B2B` | Average sales per order item. | Money. |
| `salesAndTrafficByDate[].salesByDate.averageUnitsPerOrderItem`, `…B2B` | Average units in each order item. | Number. |
| `salesAndTrafficByDate[].salesByDate.averageSellingPrice`, `…B2B` | Average selling price. | Money. |
| `salesAndTrafficByDate[].salesByDate.unitsRefunded`, `…B2B` | Number of units refunded in the period. | Count. `…B2B` not in the schema but in the file. |
| `salesAndTrafficByDate[].salesByDate.refundRate`, `…B2B` | Refund rate. | Percentage; can pass 100, as refunds can outnumber that day's orders. |
| `salesAndTrafficByDate[].salesByDate.claimsGranted`, `…B2B` | Number of A-to-z guarantee claims granted. | Count. |
| `salesAndTrafficByDate[].salesByDate.claimsAmount`, `…B2B` | Amount of claims. | Money. |
| `salesAndTrafficByDate[].salesByDate.shippedProductSales`, `…B2B` | Shipped product sales. | Money. |
| `salesAndTrafficByDate[].salesByDate.unitsShipped`, `…B2B` | Number of units shipped in the period. | Count. |
| `salesAndTrafficByDate[].salesByDate.ordersShipped`, `…B2B` | Number of orders shipped in the period. | Count. |
| `salesAndTrafficByDate[].trafficByDate.browserPageViews`, `…B2B` | Times users visited your pages in a browser. | Count. |
| `salesAndTrafficByDate[].trafficByDate.mobileAppPageViews`, `…B2B` | Times users visited your pages in the mobile app. | Count. |
| `salesAndTrafficByDate[].trafficByDate.pageViews`, `…B2B` | Times users visited your pages, browser or app. | Count. |
| `salesAndTrafficByDate[].trafficByDate.browserSessions`, `…B2B` | Browser visits; all activity within 24 hours is one session. | Count. |
| `salesAndTrafficByDate[].trafficByDate.mobileAppSessions`, `…B2B` | Mobile app visits; all activity within 24 hours is one session. | Count. |
| `salesAndTrafficByDate[].trafficByDate.sessions`, `…B2B` | Visits by browser or app; all activity within 24 hours is one session. | Count. |
| `salesAndTrafficByDate[].trafficByDate.buyBoxPercentage`, `…B2B` | Percentage of page views where the Buy Box appeared for your product. | A percentage, not a fraction. |
| `salesAndTrafficByDate[].trafficByDate.orderItemSessionPercentage`, `…B2B` | Order items relative to the people who viewed the products. | Percentage. |
| `salesAndTrafficByDate[].trafficByDate.unitSessionPercentage`, `…B2B` | Units bought relative to the people who viewed the products. | Percentage. |
| `salesAndTrafficByDate[].trafficByDate.averageOfferCount` | Average number of offers listed for sale in the period. | Total offers divided by days in the period. |
| `salesAndTrafficByDate[].trafficByDate.averageParentItems` | Average number of parent items listed for sale in the period. | |
| `salesAndTrafficByDate[].trafficByDate.feedbackReceived`, `…B2B` | Number of customer feedback received in the period. | Count. `…B2B` not in the schema but in the file. |
| `salesAndTrafficByDate[].trafficByDate.negativeFeedbackReceived`, `…B2B` | Number of negative feedback received in the period. | As above. |
| `salesAndTrafficByDate[].trafficByDate.receivedNegativeFeedbackRate`, `…B2B` | Orders with negative feedback divided by orders in the period. | As above. |
| `salesAndTrafficByAsin[].parentAsin` | ASIN of the parent product. | Equals `childAsin` for products without variations. |
| `salesAndTrafficByAsin[].childAsin` | ASIN of the child (sellable) product. | **Not unique in a period:** a child ASIN can be listed once under each of its parents, with its figures (units, sales, page views, sessions) split across the entries. Add up a child ASIN's entries before keying on it. |
| `salesAndTrafficByAsin[].salesByAsin.unitsOrdered`, `…B2B` | Number of units ordered. | Count, whole period. |
| `salesAndTrafficByAsin[].salesByAsin.orderedProductSales`, `…B2B` | Ordered product sales. | Money, whole period. **Includes VAT:** it matches `orders`' VAT-inclusive `item-price` to within 1p on nearly every ASIN-month whose units agree, with a child ASIN's entries added up. Group `orders` by UK-time month, as Business Reports does; by UTC month the units agree less often. `seller_economics` leaves VAT out from late August 2025, so don't mix the two. |
| `salesAndTrafficByAsin[].salesByAsin.totalOrderItems`, `…B2B` | Number of order items ordered. | Count, whole period. |
| `salesAndTrafficByAsin[].salesByAsin.shippedProductSales`, `…B2B` | Not in Amazon's docs for the by-ASIN section; by name, shipped product sales. | Money. `0` on every entry before January 2026 (see Traps). |
| `salesAndTrafficByAsin[].salesByAsin.unitsShipped`, `…B2B` | Not in Amazon's docs for the by-ASIN section; units shipped. | Count. `0` on every entry before January 2026. |
| `salesAndTrafficByAsin[].salesByAsin.ordersShipped`, `…B2B` | Not in Amazon's docs for the by-ASIN section; orders shipped. | Count. `0` on every entry before January 2026. |
| `salesAndTrafficByAsin[].salesByAsin.unitsRefunded`, `…B2B` | Not in Amazon's docs for the by-ASIN section; units refunded. | Count. `0` on every entry before December 2025; **partial in December 2025** (only refunds from 17 December on); complete from January 2026. |
| `salesAndTrafficByAsin[].salesByAsin.refundRate`, `…B2B` | Not in Amazon's docs for the by-ASIN section; refund rate. | Percentage. `0` on every entry before December 2025, partial in December 2025, like `unitsRefunded`. Can pass 100. |
| `salesAndTrafficByAsin[].trafficByAsin.browserSessions`, `…B2B` | Browser sessions. | Count. |
| `salesAndTrafficByAsin[].trafficByAsin.mobileAppSessions`, `…B2B` | Mobile app sessions. | Count. |
| `salesAndTrafficByAsin[].trafficByAsin.sessions`, `…B2B` | Sessions, browser or app. | Count. |
| `salesAndTrafficByAsin[].trafficByAsin.browserSessionPercentage`, `…B2B` | Share of browser sessions with at least one view of this ASIN. | Percentage. |
| `salesAndTrafficByAsin[].trafficByAsin.mobileAppSessionPercentage`, `…B2B` | Share of app sessions with at least one view of this ASIN. | Percentage. |
| `salesAndTrafficByAsin[].trafficByAsin.sessionPercentage`, `…B2B` | Share of all sessions with at least one view of this ASIN. | Percentage. |
| `salesAndTrafficByAsin[].trafficByAsin.browserPageViews`, `…B2B` | Browser page views. | Count. |
| `salesAndTrafficByAsin[].trafficByAsin.mobileAppPageViews`, `…B2B` | Mobile app page views. | Count. |
| `salesAndTrafficByAsin[].trafficByAsin.pageViews`, `…B2B` | Page views, browser or app. | Count. |
| `salesAndTrafficByAsin[].trafficByAsin.browserPageViewsPercentage`, `…B2B` | This ASIN's share of browser page views. | Percentage. |
| `salesAndTrafficByAsin[].trafficByAsin.mobileAppPageViewsPercentage`, `…B2B` | This ASIN's share of mobile page views. | Percentage. |
| `salesAndTrafficByAsin[].trafficByAsin.pageViewsPercentage`, `…B2B` | This ASIN's share of all page views. | Percentage. |
| `salesAndTrafficByAsin[].trafficByAsin.buyBoxPercentage`, `…B2B` | Percentage of page views where the Buy Box appeared for your product. | Percentage, 0 to 100. |
| `salesAndTrafficByAsin[].trafficByAsin.unitSessionPercentage`, `…B2B` | Units ordered divided by sessions. | Percentage; can pass 100 when units outnumber sessions. |

Documented but not in the file: `salesAndTrafficByAsin[].sku` (only with `SKU` granularity). The B2B fields Amazon's report page lists but its schema lacks (`shippedProductSalesB2B`, `unitsShippedB2B`, `ordersShippedB2B`, `unitsRefundedB2B`, `refundRateB2B`, `claimsGrantedB2B`, `claimsAmountB2B`, `feedbackReceivedB2B`, `negativeFeedbackReceivedB2B`, `receivedNegativeFeedbackRateB2B`) sit beside their non-B2B fields in `salesByDate` and `trafficByDate`.

Source: [sellerSalesAndTrafficReport.json schema](https://github.com/amzn/selling-partner-api-models/blob/main/schemas/reports/sellerSalesAndTrafficReport.json) and [Analytics Reports: Sales and Traffic Business Report](https://developer-docs.amazon/sp-api/docs/report-type-values-analytics); the sales formula from [Using the different types of Business reports (UK help)](https://sellercentral.amazon.co.uk/help/hub/reference/external/G202142060).

## Downloaded from Seller Central

**A different file from the API's.** The API sends one JSON document with a by-date and a by-ASIN section. Seller Central gives one CSV per report page, formatted for display. Seen in a real UK seller's Seller Central downloads (Oct 2026), for one month by day (By Date > Sales and traffic) and the same month by child item (By ASIN > Detail page sales and traffic by child item):

- **File name:** `BusinessReport-DD-MM-YYYY.csv`, named by the day you download it, not the period it covers. Rename it when you save it.
- **Format:** comma-separated, UTF-8 with a byte order mark, with quotes only around values that need them. The last line has no line ending.
- **Column names** use an en dash (`–`) and mixed capitals (`Ordered Product Sales` beside `Ordered product sales – B2B`). Match them exactly, or normalise case and dashes first.
- **Numbers are display text:** money as `£1,234.56` (pound sign and thousands comma), counts of 1,000 or more as `"1,234"`, percentages as `12.34%` (0 to 100, as in the API). Strip `£`, `,` and `%` before doing sums.
- **Dates** in the `Date` column are DD/MM/YYYY.
- **The columns follow the screen.** The page has a Show/Hide columns tab, and the download seems to follow it, so a file can have other columns than those below. Find columns by name.

By date, 18 columns: `Date`, `Ordered Product Sales`, `Ordered product sales – B2B`, `Units ordered`, `Units ordered – B2B`, `Total order items`, `Total order items – B2B`, `Average sales per order item`, `Average sales per order item – B2B`, `Average units per order item`, `Average units per order item – B2B`, `Average selling price`, `Average selling price – B2B`, `Sessions – Total`, `Sessions – Total – B2B`, `Order Item Session Percentage`, `Order item session percentage – B2B`, `Average offer count`. By name they are the `salesByDate` and `trafficByDate` fields of the same names. The file had no refund, claim, shipped, page view, Buy Box or feedback columns, and no browser and app split.

By child item, 21 columns: `(Parent) ASIN`, `(Child) ASIN`, `Title`, `Sessions – Total`, `Sessions – Total – B2B`, `Session percentage – Total`, `Session percentage – Total – B2B`, `Page views – Total`, `Page views – Total – B2B`, `Page views percentage – Total`, `Page views percentage – Total – B2B`, `Featured Offer (Buy Box) percentage`, `Featured Offer (Buy Box) percentage – B2B`, `Units ordered`, `Units ordered – B2B`, `Unit Session Percentage`, `Unit session percentage – B2B`, `Ordered Product Sales`, `Ordered product sales – B2B`, `Total order items`, `Total order items – B2B`. There is no date column: each row covers the whole period, like `salesAndTrafficByAsin`. **No SKU column**, totals only (no browser and app split), and no refund or shipped columns. The Growth opportunities column on the screen is not in the file.

Custom analytics, the page Amazon is moving sellers to, gave an .xlsx file, not a CSV, when a chart was exported. For traps shared by all Seller Central downloads, see [seller-central-downloads](seller-central-downloads.md).

## Personal data

None. The figures are totals per store and per ASIN, with no buyer details.

## Questions it answers

- What were each store's daily ordered sales, units and sessions?
- Which child ASINs get page views but few orders (low unit session percentage)?
- What share of page views had the Buy Box for each ASIN, per store per month?
- Which ASINs and stores have the most refunded units, from January 2026 on? Before that the per-ASIN refund fields are all `0`, or in December 2025 hold only refunds from the 17th; `salesByDate.unitsRefunded` has refunds per store per day in every month.
- How many feedback and negative feedback came in per day?

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Days are UK time, not UTC.** Each day's `unitsOrdered` in the UK store matched `orders`' units by UK-time purchase date on every full day checked, but often not by UTC date. Convert `orders`' `purchase-date` (UTC) to UK time before comparing, by day or by month.
- **A file can stop a day short.** Asked for a month up to yesterday, one store's file can reach that day while another store's stops a day earlier. Take a file's last day from its last `date`, not from the period you asked for.
- **An ASIN entry isn't a sale.** A `salesAndTrafficByAsin` entry can have page views but no orders, and a store can have entries in a month with no orders at all. Count a store as selling when its by-date `unitsOrdered` is above `0`. Every store's file has a full `salesAndTrafficByDate`, one entry per day, even with no sales.
- **Per-ASIN shipped and refund fields start late.** In `salesByAsin`, `unitsRefunded` and `refundRate` are `0` on every entry before December 2025. In December 2025 the per-ASIN refunds equal the by-date refunds from 17 December on exactly, so the month is partial. From January 2026 they equal the by-date total every month. `unitsShipped`, `ordersShipped` and `shippedProductSales` are `0` on every entry before January 2026. The by-date fields show shipments and refunds in every month, so these zeros mean not reported per ASIN, not none.
- **A child ASIN can have several entries** in one month, one per parent, with its figures split between them (see `childAsin`).
- **Percentages are 0 to 100,** but `unitSessionPercentage` and `refundRate` can pass 100.
- **Not throttled** when asked store by store, a month at a time, back to Amazon's two-year limit.

Seen in Seller Central downloads (Oct 2026):

- **The CSVs hold display text, not numbers.** Money carries `£` and thousands commas, larger counts are quoted with a comma, and percentages end in `%`, so a spreadsheet or script can read them as text. And every file is named by its download date, so two months downloaded on the same day get the same name.
