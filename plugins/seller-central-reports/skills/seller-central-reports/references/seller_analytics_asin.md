# Seller Analytics, per ASIN (`seller_analytics_asin`)

Amazon's Seller Analytics data per ASIN per store for one calendar month (sales, traffic, promotions, reviews and returns), from the same Data Kiosk dataset as [seller_analytics](seller_analytics.md) asked without account totals, account health or defects, because asking for those makes Amazon add sales and traffic up per store.

## Where to find it in Seller Central

Not confirmed: no single Seller Central screen shows this dataset, and no Amazon page links it to one. The sales and traffic columns are the measures Business reports shows under **By ASIN > Detail page sales and traffic by child item**, but that is the `sales_and_traffic` report's screen, and the figures can differ (see Traps). Business reports is not in the new main menu: open **Site map** (at the bottom of the main menu) > **Other resources** > **Business reports**. Classic: **Reports > Business Reports**. Checked in Seller Central UK on 28 Oct 2026. A banner there then asked sellers to switch to Custom Analytics by year-end.

## Getting it through the API

Same dataset as [seller_analytics](seller_analytics.md) ([Seller Analytics guide](https://developer-docs.amazon/sp-api/docs/seller-analytics-dataset-guide)). It overlaps `sales_and_traffic`, which also has units, sales, page views, sessions and Featured Offer (Buy Box) percentage per child ASIN per store per month. This one adds promotion (deal) details and returns per ASIN, and covers every store in one request.

- **Call / report type:** Data Kiosk 2023-11-15, as for `seller_analytics`: wait until no query is `IN_QUEUE` or `IN_PROGRESS`, send `createQuery` with the query text, poll `getQuery` until it is done, then `getDocument`, following `pagination.nextToken`.
- **Query:** `analytics_sellerAnalytics_2025_03_31.sellerAnalyticsView` with `startDate` and `endDate` set to the calendar month, `aggregateBy: MONTH`, `currencyCode: "GBP"` and `marketplaceId` set to every store wanted. `groupByKey`: `asin`, `parentAsin`, `productTitle`, `brand`, `brandCode`, `productGroup`, `marketplaceId`. Per row: the `sales`, `traffic`, `promotion`, `search`, `reviews` and `returns` groups, with the same fields as `seller_analytics`' per-row groups.
- **Left out on purpose:** `totals`, `accountHealth` and `defects` (they make Amazon add up per store), plus everything `seller_analytics` leaves out (B2B metrics, `timeRangeBenchmarking`, `msku`, `fulfilmentChannel`, the sales-and-lending fields).
- **Role:** as for `seller_analytics`: Brand Analytics for Data Kiosk, and "sales and traffic access" for the fields ([role mappings](https://developer-docs.amazon/sp-api/docs/role-mappings), [guide](https://developer-docs.amazon/sp-api/docs/seller-analytics-dataset-guide)).
- **How far back Amazon goes:** `startDate` no more than two years ago ([guide](https://developer-docs.amazon/sp-api/docs/seller-analytics-dataset-guide)). The query has no snapshot metrics, so the 24-month rule for those doesn't apply.
- **When a month is ready:** as for `seller_analytics`. Returns come in about 45 days late.

## Columns

JSON Lines. The whole month is one line: `{startDate, endDate, metrics[]}`, one entry in `metrics` per ASIN per store. If Amazon splits a result into pages, join the pages in order. Field paths as seen in a real UK seller's files (Oct 2026), every month checked; formats only, no figures.

Shapes, as in `seller_analytics`: **count** or **rate** is `<metric>.value`; **money** is `<metric>.amount.amount` plus `.amount.currencyCode` (the currency asked for, e.g. `GBP`); **units + money** is `<metric>.unitsWithValue.units` plus `.unitsWithValue.value.amount` and `.currencyCode`. Rates are fractions, even those named `…Pct`, from 0 to 1, bar an occasional `conversionRatePct` above 1 and `promotionDiscountPct`, which isn't a 0 to 1 fraction at all (see the deal details below).

| Column | Meaning | Format and traps |
|---|---|---|
| `startDate`, `endDate` | The month. | `YYYY-MM-DD`. |
| `metrics[].groupByKey.asin` | The ASIN (child). | One row per ASIN per store. |
| `metrics[].groupByKey.parentAsin` | Its parent ASIN. | Often equal to `asin`. |
| `metrics[].groupByKey.productTitle` | The listing title. | |
| `metrics[].groupByKey.brand` | Brand name. | Can be `null`. |
| `metrics[].groupByKey.brandCode` | Amazon's brand code. | A number written as text, usually with `.0` on the end; sometimes in scientific notation, or an empty string (always with a `null` brand). |
| `metrics[].groupByKey.productGroup` | Amazon's product group. | |
| `metrics[].groupByKey.marketplaceId` | The store. | e.g. `A1F83G8C2ARO7P` (UK). |
| `metrics[].metrics.sales.unitsOrdered`, `unitsShipped` | Count. | Can be `0`: an ASIN with traffic but no sales still has a row. |
| `metrics[].metrics.sales.orderedProductSales`, `orderGMS`, `averageSellingPrice` | Money. Price times units ordered; order value after promotions, excluding cancelled orders; average price per unit. | **`orderedProductSales` includes VAT:** its UK total usually matches `sales_and_traffic`'s by-date total very closely, and `sales_and_traffic` matches `orders`' VAT-inclusive `item-price`. It can run above `sales_and_traffic` (see Traps). |
| `metrics[].metrics.sales.ordersShippedWithRevenue` | Units + money. | |
| `metrics[].metrics.traffic.pageViews`, `sessions`, `browserPageViews`, `browserSessions`, `mobileAppPageViews`, `mobileAppSessions`, `glanceViews`, `featureOfferGlanceViews` | Count. | **The browser figures are far too low,** as in `seller_analytics`. Amazon's schema says `pageViews` is the sum of `browserPageViews` and `mobileAppPageViews`, and `sessions` likewise, but on most rows with page views browser plus app falls short. `pageViews`, `sessions` and the app fields equal `sales_and_traffic`'s on nearly every ASIN with page views; the browser fields rarely do, and come out well below Business Reports'. **Use `pageViews` and `sessions` as they are, and take the browser vs app split from `sales_and_traffic`.** |
| `metrics[].metrics.traffic.featureOfferPct` | Rate. Share of glance views where the seller's offer was the Featured Offer (Buy Box). | From `0.0` to `1.0`. `0` exactly where `featureOfferGlanceViews` is `0`, which includes rows with glance views where the seller's offer was never the Featured Offer. |
| `metrics[].metrics.traffic.conversionRatePct` | Rate. | A fraction, despite `Pct`, but can pass 1 where more units sold than there were sessions. |
| `metrics[].metrics.promotion.promotionOrderedUnitsWithRevenue`, `promotionDiscountAmt` | Units + money; money. | `0` except on rows with a deal. |
| `metrics[].metrics.promotion.promotionConversionRate`, `promotionGlanceViewCnt` | Rate, count. | Can be `0` on every row, so don't rely on it. |
| `metrics[].metrics.promotion.promotionStartDate`, `promotionEndDate`, `promotionType`, `promotionStatus`, `promotionAsinStateType`, `promotionDiscountPct`, `promotionOrderedPrice`, `dealId`, `dealTitle`, `dealOurPrice` | One deal's details. | `null` except on rows with a deal, matching `seller_analytics`' deal rows (the same ASINs and `dealId`s). Dates as `YYYY-MM-DD`; an end date can be years ahead. **`promotionDiscountPct` isn't a 0 to 1 fraction:** it's usually negative and can be below −1 (`0` where `dealOurPrice` is `0`). It's `promotionDiscountAmt` ÷ `dealOurPrice`, as Amazon defines it, but `promotionDiscountAmt` has no fixed scale: sometimes one unit's discount (`promotionOrderedPrice` − `dealOurPrice`, so negative), sometimes that times the deal's units (`promotionOrderedUnitsWithRevenue`), sometimes neither. For a deal's discount use neither: compare `dealOurPrice` (before the deal) with `promotionOrderedPrice` (after it, and lower). Both look to leave VAT out, unlike the sales figures: the deal's revenue per unit can be exactly 1.2 times `promotionOrderedPrice`, and `orders`' highest UK unit price for the ASIN that month tends to be about 1.2 times `dealOurPrice` or more. |
| `metrics[].metrics.search` | The only per-row search fields asked for: `catalog.clickRate`, `conversionRate`, `trafficSales`. | Can be `null` on every row. |
| `metrics[].metrics.reviews.lowRatingReviewCount`, `totalReviewCount`, `criticalReviewRate` | Count, count, rate. Reviews received in the month. | Can be `0` on every row, so don't rely on them. |
| `metrics[].metrics.reviews.displayRating`, `rawRating`, `totalRatingCount` | Star rating and the ratings behind it. | Can be `null` on every row. `seller_analytics` may not give rating counts either: its `totalRatingCount` can be `0`. |
| `metrics[].metrics.returns.returnedUnits` | Count. Units returned with a refund or replacement, including returnless refunds, counted in the month each unit was **shipped**, not the month it came back ([schema](https://github.com/amzn/selling-partner-api-models/blob/main/schemas/data-kiosk/analytics_sellerAnalytics_2025_03_31.graphql)). So it won't match `fba_returns`, which is dated by return. | `null` exactly where `unitsShipped` is `0`, else a count (can be `0`). Amazon says it may be null or incomplete for the most recent ~45 days. |
| `metrics[].metrics.returns.totalRefundAmount` | Money. | `null` where no refund is reported; filled where Amazon reports one, and like `returnedUnits` may be null or incomplete for the most recent ~45 days. A refund can come in a month with no returned units on that row, and returned units with no refund. |
| `metrics[].metrics.returns.returnedRate` | Rate. Returned units over shipped units. | About 45 days late, so recent months may be low. `0`, not `null`, where `returnedUnits` is `null`. Usually equal to `returnedUnits` ÷ `unitsShipped`, sometimes lower. |

Source: [Seller Analytics Dataset Use Case Guide](https://developer-docs.amazon/sp-api/docs/seller-analytics-dataset-guide) and the schema [analytics_sellerAnalytics_2025_03_31.graphql](https://github.com/amzn/selling-partner-api-models/blob/main/schemas/data-kiosk/analytics_sellerAnalytics_2025_03_31.graphql).

## Personal data

None. The rows are per ASIN per store, with no buyer or business details.

## Questions it answers

- Which ASINs, in which stores, got plenty of glance views but a low Featured Offer percentage last month?
- How did units, ordered sales and average selling price move month by month for an ASIN across the stores?
- Which ASINs have a high returned rate or refund total?
- Which ASINs ran a deal in a month, and how many units sold on it?
- Which ASINs get page views but no sales (high `pageViews`, `unitsOrdered` 0)?

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Rows without sales.** Rows can have `unitsOrdered` 0: Amazon lists every ASIN with traffic, not only those that sold, and some rows are zero or `null` in every metric. A store can have a row in a month with no sale in it (shipments of earlier orders only, a refund only, or all zero).
- **Browser page views and sessions are far too low** in every month, so browser plus app rarely adds up to `pageViews`. `pageViews`, `sessions` and the app fields match `sales_and_traffic`; take the browser vs app split from there.
- **Seller Analytics and Business Reports don't always agree.** This query's units and sales for a store add up to `seller_analytics`' store row. With each child ASIN's `sales_and_traffic` entries added up (a child ASIN can be listed once under each of its parents there), they usually match Business Reports too, but they can run above Business Reports while `orders` agrees with Business Reports. The cause wasn't found. Check against `sales_and_traffic` and `orders` before relying on either.
- **Fields that can come back empty:** `search` and the star ratings (`displayRating`, `rawRating`, `totalRatingCount`) can be `null` on every row, and the review counts, `promotionConversionRate` and `promotionGlanceViewCnt` can be `0`. Don't treat those zeros as real figures.
- **Returns are partly null.** `returnedUnits` is `null` wherever nothing shipped, and `totalRefundAmount` on rows with no refund reported. Don't read `null` as zero: Amazon says both "may be null or incomplete" for the most recent ~45 days, and that a null in a grouped row means "not available at this granularity" ([guide](https://developer-docs.amazon/sp-api/docs/seller-analytics-dataset-guide)). Treat a month that ended under 45 days ago as provisional and ask for it again later.
