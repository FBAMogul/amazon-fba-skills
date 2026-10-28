# Seller Analytics, per store and account (`seller_analytics`)

Amazon's Seller Analytics data for one calendar month, asked for with account totals, account health and defects, which makes Amazon send sales, traffic, promotions, account health, reviews and returns per store (not per ASIN), return-defect topics and deal details per ASIN, and account-wide totals; for the same metrics per ASIN per store, use [seller_analytics_asin](seller_analytics_asin.md).

## Where to find it in Seller Central

Not confirmed: no single Seller Central screen shows this dataset, and no Amazon page links it to one. Seller Central's own sales and traffic screens are **Business reports** and **Custom analytics**. Neither is in the new main menu: open **Site map** (at the bottom of the main menu) > **Other resources**. Classic: **Reports > Business Reports** and **Reports > Custom analytics**. Checked in Seller Central UK on 28 Oct 2026. A banner on Business reports then asked sellers to switch to Custom Analytics by year-end.

## Getting it through the API

Seller Analytics is a Data Kiosk dataset with "more than 100 metrics" across sales, traffic, promotion, account health, search, reviews, defects and returns ([Seller Analytics guide](https://developer-docs.amazon/sp-api/docs/seller-analytics-dataset-guide)). Amazon added it in its September 2026 release ([release notes](https://developer-docs.amazon/sp-api/docs/sp-api-release-notes)).

- **Call / report type:** Data Kiosk 2023-11-15, a GraphQL query instead of a report type.
  - Amazon allows only one pending request per query, seller and app ([Data Kiosk API](https://developer-docs.amazon/sp-api/docs/data-kiosk-api)), so wait until `getQueries` shows no query `IN_QUEUE` or `IN_PROGRESS`.
  - Send `createQuery` with the query text. A query must be at most 8000 characters after whitespace is removed ([createQuery](https://developer-docs.amazon/sp-api/reference/createquery)).
  - The query behind the columns below calls `analytics_sellerAnalytics_2025_03_31.sellerAnalyticsView` with `startDate` and `endDate` set to the calendar month, `aggregateBy: MONTH`, `currencyCode: "GBP"`, and `marketplaceId` set to every store wanted. It asks for rows per ASIN per store, plus `totals`, `accountHealth` and `defects`.
  - Poll `getQuery` until it is done, then download the result through `getDocument`.
  - If the result has `pagination.nextToken`, send `createQuery` again with that token and the query text Amazon returned, and join the documents in order.
  - If the query ends `DONE` with no data document, Amazon had no data for that window. If it ends `FATAL`, the query failed: fetch the error document through `getDocument` and report the failure. Never treat a `FATAL` query as no data ([Data Kiosk API](https://developer-docs.amazon/sp-api/docs/data-kiosk-api)).
- **Query depth cap:** Amazon caps a Data Kiosk query's "complexity", which is how deep its braces nest, at under 10. A query nested 11 deep was refused for every month ("The provided query is invalid"). To get under the cap, the query behind these columns keeps only the `units` of each `…WithPrice` metric under `totals.search.catalog.priceAndCountMetrics` (not their `value` amounts), and per row keeps only `search.catalog.clickRate`, `conversionRate` and `trafficSales` (no `priceAndCountMetrics` and no per-keyword `query` metrics).
- **Fields the query leaves out, and why:** the B2B metrics, which the guide says are "not yet supported"; `timeRangeBenchmarking` and `totals.search.query`, to stay under the 8000-character limit; `lending`, `sellerAttributes` and `encryptedMerchantCustomerId`, which need sales and lending access; `msku` and `fulfilmentChannel`, which would split an ASIN's rows; and the deprecated `conversationRatePct`.
- **Role:** every Data Kiosk operation needs [Brand Analytics](https://developer-docs.amazon/sp-api/docs/role-mappings). Data Kiosk also checks each field in the query against the app's roles when the query is created ([Data Kiosk API](https://developer-docs.amazon/sp-api/docs/data-kiosk-api)). The guide says every group this query asks for needs "sales and traffic access", but it doesn't name the SP-API role that grants it ([guide](https://developer-docs.amazon/sp-api/docs/seller-analytics-dataset-guide)). Asking for a field the app isn't authorised for returns an authorisation error, not a null, so a first test query shows whether your app has this access.
- **How far back Amazon goes:** `startDate` can be no more than two years ago. Because the query includes the 120-day snapshot metrics, a monthly request can also reach back only to "the current month and the previous 23 months" ([guide](https://developer-docs.amazon/sp-api/docs/seller-analytics-dataset-guide)).
- **When a month is ready** ([guide](https://developer-docs.amazon/sp-api/docs/seller-analytics-dataset-guide)): with `MONTH`, an `endDate` that isn't a month's last day moves back a month; `endDate` is capped at the last day Amazon has data for; and a day's data is generally ready two days later. So ask for a month only once it is over and its last days are ready. Return data is computed "with a delay of about 45 days", so a month's returns keep changing for about that long.

## Columns

JSON Lines. Field paths as seen in a real UK seller's files (Oct 2026), every month checked; formats only, no figures.

**What the file really holds.** The whole month is **one JSONL line**: `{startDate, endDate, totals, metrics[]}`. [Amazon](https://developer-docs.amazon/sp-api/docs/data-kiosk-api) says Data Kiosk returns JSONL but doesn't say what one line holds. The query asks for rows per ASIN per store, but Amazon returns:

- **One store row per store asked for,** with only `groupByKey.marketplaceId` filled (`asin`, `parentAsin`, `productTitle`, `brand`, `brandCode` and `productGroup` are `null`).
  - Stores with any activity that month hold `sales`, `traffic`, `promotion`, `accountHealth`, `returns` and `reviews`. Some stores with no activity hold those groups too, zero in every sales, traffic, returns and promotion field.
  - **A row with the groups doesn't mean the store sold.** Some have only page views, only shipments of earlier orders, or only a refund. Count a store as selling when `sales.unitsOrdered` is above `0`. `sales_and_traffic` agrees on which stores had orders.
  - The other store rows hold only the seven `…T120Days` account-health snapshots, with the rest `null`.
- **ASIN rows,** of two kinds:
  - **Return-defect rows,** one per ASIN with a return-defect topic, holding `defects` and `reviews.totalRatingCount` (which can be `0`). Their values are identical in every month: they're a snapshot taken when the query ran, not that month's data.
  - **Deal rows,** one per ASIN on a deal that month, holding `promotion` with the deal's details. An ASIN on a deal that also has a return-defect topic gets one row with both, so count deals by `promotion.dealId`, not by rows.
  - `sales`, `traffic`, `accountHealth`, `search` and `returns` are `null` on every ASIN row.

So in this query, sales, traffic and returns are **per store, not per ASIN**. For per-ASIN sales and traffic use [seller_analytics_asin](seller_analytics_asin.md) or `sales_and_traffic` (per child ASIN per store per month); both include VAT in sales, as this one does. `seller_economics` has sales per SKU, but leaves VAT out from late August 2025, so don't mix it with them.

**Page views:** per store per month in the `traffic` group (`pageViews`, `browserPageViews`, `mobileAppPageViews`). Use `pageViews` and `sessions` as they are, but not `browserPageViews` or `browserSessions`, which are far too low (see below): take the browser vs app split from `sales_and_traffic`.

Shapes: **count** or **rate** is `<metric>.value`; **money** is `<metric>.amount.amount` plus `.amount.currencyCode` (the currency asked for, e.g. `GBP`); **units + money** is `<metric>.unitsWithValue.units` plus `.unitsWithValue.value.amount` and `.currencyCode`. Rates are fractions from 0 to 1, even those named `…Pct`, bar `promotionDiscountPct`, which isn't a 0 to 1 fraction at all (see the deal rows below).

| Column | Meaning | Format and traps |
|---|---|---|
| `startDate`, `endDate` | The month. | `YYYY-MM-DD`. |
| `totals.sales.averageSellingPrice`, `orderedProductSales`, `orderGMS` | Money. Average price per unit; price times units sold; order value after promotions, excluding cancelled orders. | Whole account, all stores asked for: `orderedProductSales` is the sum of the store rows. **Includes VAT** (see `metrics[].metrics.sales.*`). `orderGMS` can include cancelled orders: a store row can have a non-zero `orderGMS` and no units ordered when that store's orders that month were cancelled. |
| `totals.sales.ordersShippedWithRevenue` | Units + money. Orders shipped and their revenue. | |
| `totals.sales.unitsOrdered`, `unitsShipped` | Count. | |
| `totals.traffic.browserPageViews`, `browserSessions`, `mobileAppPageViews`, `mobileAppSessions`, `pageViews`, `sessions`, `glanceViews`, `featureOfferGlanceViews` | Count. Page views, sessions and glance views. | **The browser figures are far too low.** Amazon's schema says `pageViews` is the sum of `browserPageViews` and `mobileAppPageViews`, and `sessions` likewise, but they don't add up: browser plus app falls short. `pageViews`, `sessions` and the two `mobileApp…` fields match Business Reports (`sales_and_traffic`); the browser ones come out well below it, so the browser share of page views looks much smaller than it is. **Use `pageViews` and `sessions` as they are, and take the browser vs app split from `sales_and_traffic`.** |
| `totals.traffic.conversionRatePct`, `featureOfferPct` | Rate. Search-click conversion; share of glance views where the offer was the Featured Offer. | Fractions, despite `Pct`. |
| `totals.promotion.promotionConversionRate`, `promotionGlanceViewCnt` | Rate, count. | Can be `0` in every month, so don't rely on it. |
| `totals.promotion.promotionOrderedUnitsWithRevenue`, `promotionDiscountAmt` | Units + money; money. | |
| `totals.promotion.promotionOrderedPrice`, `promotionDiscountPct`, `promotionStartDate`, `promotionEndDate`, `promotionAsinStateType`, `promotionType`, `promotionStatus`, `dealOurPrice`, `dealId`, `dealTitle` | Per-promotion details. | In totals, filled in some months and `null` in others. Always `null` on store rows. |
| `totals.accountHealth.accountHealth…Count`, `…Rate`, `accountHealthRatingScore`, `accountHealthTotalOrderDefects` | The 21 account-health counts, rates and the Account Health Rating score, whole account. | `accountHealthRatingScore` is `0` in every month, here and on every store row that has it, though `account_health` has a real score. Use `account_health` for the score. |
| `totals.search.catalog.priceAndCountMetrics.impressionsWithPrice`, `clicksWithPrice`, `cartAddsWithPrice`, `purchasesWithPrice` | Units only (`unitsWithValue.units`). Search impressions, clicks, adds to basket and purchases. | No `value` amounts: trimmed to meet Amazon's query depth cap. Can all be `0` in every month, so don't rely on them. |
| `totals.search.catalog.priceAndCountMetrics.sameDayMetrics`, `oneDayMetrics`, `twoDayMetrics` | Count. `shippingImpression`, `shippingClick`, `shippingCartAdd`, `shippingPurchase` where that delivery speed was offered. | Can all be `0` in every month, so don't rely on them. |
| `totals.search.catalog.clickRate`, `conversionRate` | Rate. | Can be `0.0` in every month, so don't rely on it. |
| `totals.search.catalog.trafficSales` | Money. Revenue from search traffic. | Can be `0` in every month, so don't rely on it. |
| `totals.reviews.lowRatingReviewCount`, `totalReviewCount` | Count. Reviews received in the month. | Can be `0` in every month, so don't rely on it. |
| `totals.returns.returnedUnits`, `totalRefundAmount` | Count; money. `returnedUnits` counts units returned with a refund or replacement, including returnless refunds, in the month each unit was shipped, not the month it came back. | About 45 days late, so recent months may be low. Won't match `fba_returns`, which is dated by return. |
| `metrics[].groupByKey.marketplaceId` | The store the row belongs to. | e.g. `A1F83G8C2ARO7P` (UK). Filled on every row. |
| `metrics[].groupByKey.asin`, `parentAsin`, `productTitle`, `brand`, `brandCode`, `productGroup` | Catalogue details. | `null` on the store rows; filled on the ASIN rows. |
| `metrics[].metrics.sales.*`, `traffic.*`, `promotion.*` | As in `totals`, for one store. | Store rows of the stores with any activity that month, plus some all-zero rows (see above); a store sold only where `unitsOrdered` is above `0`. On store rows the per-promotion details are `null`, and `promotionConversionRate` and `promotionGlanceViewCnt` are `0`. **Browser page views and sessions are far too low,** as in `totals`: browser plus app never equalled `pageViews` on a store row with page views. `pageViews`, `sessions` and the app fields usually equal `sales_and_traffic`'s; the browser fields only on rows with no page views. **Sales include VAT:** UK `orderedProductSales` usually matches Business Reports' by-date total (`sales_and_traffic`, which matches `orders`' VAT-inclusive `item-price`) very closely, but can run above it for months at a time, for reasons not found. |
| `metrics[].metrics.promotion.promotionStartDate`, `promotionEndDate`, `promotionType`, `promotionStatus`, `promotionAsinStateType`, `promotionDiscountPct`, `promotionOrderedPrice`, `dealId`, `dealTitle`, `dealOurPrice` | One deal's details. | Only on the deal ASIN rows. **`promotionDiscountPct` isn't a 0 to 1 fraction:** it's usually negative, on deal rows and in totals, and sometimes below −1. In totals it's `promotionDiscountAmt` ÷ `dealOurPrice`, as Amazon defines it, and `promotionDiscountAmt` has no fixed scale (see [seller_analytics_asin](seller_analytics_asin.md)); on these rows `promotionDiscountAmt` is `null`. For a deal's discount, compare `dealOurPrice` with `promotionOrderedPrice`, which equal `seller_analytics_asin`'s on every deal row; both look to leave VAT out. |
| `metrics[].metrics.accountHealth.accountHealth…` | As in `totals`, for one store. | Same store rows as `sales`. Rates are fractions from 0 to 1. `accountHealthRatingScore` is `0`, as in totals. |
| `metrics[].metrics.accountHealth.regionResolvedDefectsUnitT120Days`, `regionIncorrectLabelDefectRateT120Days`, `regionLabelMissingDefectRateT120Days`, `regionInaccurateProductDefectRateT120Days`, `regionUnexpectedProductDefectRateT120Days`, `regionOtherDefectsProductRelatedDefectRateT120Days`, `regionDistinctCountDefectTypeT120Days` | Trailing 120-day product defect figures for the whole region (Amazon calls them snapshots). | On every store row, not per ASIN, with the same value on every row: one region-wide figure, so read one row and never add the rows up (that multiplies the real count by the number of stores). **Each month holds its own figure.** Amazon's schema says they reflect "the current state at query time", but months all queried on the same day held different values. Don't add them across months either: Amazon's guide says they "cannot be added across periods". |
| `metrics[].metrics.search.catalog.clickRate`, `conversionRate`, `trafficSales` | The only per-row search fields the trimmed query asks for. | `search` can be `null` on every row. |
| `metrics[].metrics.reviews.lowRatingReviewCount`, `totalReviewCount`, `criticalReviewRate` | Count, count, rate. | The same store rows as `sales`. Can be `0` on all of them, so don't rely on them. |
| `metrics[].metrics.reviews.displayRating`, `rawRating` | Star rating, rounded and raw. | Can be `null` on every row. |
| `metrics[].metrics.reviews.totalRatingCount` | Count. Ratings behind the star rating. | The return-defect ASIN rows only, and `0` on all of them: present, but not a real count. |
| `metrics[].metrics.defects.topics[].returnDefectTopic` | Text. The customer-reported defect topic. | Amazon's wording: a short phrase naming the defect. One topic per ASIN row. The same every month. |
| `metrics[].metrics.defects.topics[].returnDefectFrequency`, `returnDefectContribution` | Count; rate (share of the ASIN's defects). | Can both be `0` even where a topic is named, so don't rely on them. |
| `metrics[].metrics.defects.topics[].returnDefectRecommendations` | Text. Amazon's suggested fix. | Can be `null` on every row. |
| `metrics[].metrics.defects.totalDefectFrequency` | Count. | Can be `0` on every row. |
| `metrics[].metrics.returns.returnedUnits`, `totalRefundAmount`, `returnedRate` | Count; money; rate. Returns are counted in the month the unit was shipped, as in `totals`. | Store rows only. |

Not in the query behind these columns (see "Fields the query leaves out"): B2B metrics, `timeRangeBenchmarking`, `totals.search.query` and per-row `search.query`, per-row `search.catalog.priceAndCountMetrics`, the search `value` amounts in totals, `lending`, `sellerAttributes`, `encryptedMerchantCustomerId`, `msku`, `fulfilmentChannel` and the deprecated `conversationRatePct`.

Source: [Seller Analytics Dataset Use Case Guide](https://developer-docs.amazon/sp-api/docs/seller-analytics-dataset-guide) and the schema [analytics_sellerAnalytics_2025_03_31.graphql](https://github.com/amzn/selling-partner-api-models/blob/main/schemas/data-kiosk/analytics_sellerAnalytics_2025_03_31.graphql) (meanings are from the schema's field descriptions).

## Personal data

None: no buyer or business personal details. The `totals` block holds the seller's own whole-account trading figures.

## Questions it answers

Per-ASIN questions (glance views, Featured Offer %, units, returns per ASIN) need [seller_analytics_asin](seller_analytics_asin.md). This query answers:

- How did each store's units, ordered sales, glance views and Featured Offer percentage move month by month?
- How did account health metrics trend month by month, for the account and per store? For the Account Health Rating score use `account_health`: `accountHealthRatingScore` can be `0` here.
- Which ASINs have a return-defect topic, and which defects does Amazon name (as of when the query ran, not per month)?
- Which deals ran in a month, on which ASINs?

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **The query depth cap.** A query nested 11 braces deep was refused for every month with "The provided query is invalid". Trimmed to depth 9 (see "Query depth cap"), every month came back.
- **Per store, not per ASIN.** In every month, sales, traffic, promotion, account health, returns and reviews came back on one row per store; the per-ASIN rows hold only return-defect topics (with a rating count that's always `0`) and deal details. Tests found two causes: the `totals` block, and the `accountHealth` and `defects` groups asked together. Without them every group comes back per ASIN: that's [seller_analytics_asin](seller_analytics_asin.md).
- **Browser page views and sessions are far too low** in every month, in `totals` and on every store row with page views, so browser plus app doesn't add up to `pageViews`. `pageViews`, `sessions` and the app fields match `sales_and_traffic`; take the browser vs app split from there.
- **Seller Analytics and Business Reports don't always agree.** A store's `orderedProductSales` usually matches Business Reports closely, but it can run above it for months at a time, for reasons not found. Check against `sales_and_traffic` before relying on either.
- **The T120Days defect figures are per month; the defect topics aren't.** The seven T120Days fields on the store rows hold each month's own trailing 120-day figure, though Amazon's schema says they show the state at query time. Each is one region-wide value repeated on every store row, so don't add the rows up. The return-defect ASIN rows are identical in every month, so those are a snapshot as of the query.
- **Fields that can come back empty.** The search totals and per-row `search`, the defect frequencies and contributions, `accountHealthRatingScore`, the review counts (`lowRatingReviewCount`, `totalReviewCount`, `criticalReviewRate`, and `totalRatingCount` on the return-defect rows), `promotionConversionRate`, `promotionGlanceViewCnt`, `displayRating`, `rawRating` and `returnDefectRecommendations` can be `0` or `null` month after month. Don't treat a zero there as a real figure: take reviews and ratings from elsewhere, and the Account Health Rating from `account_health`.
- **The whole month is a single JSONL line,** so a line count is always 1.
