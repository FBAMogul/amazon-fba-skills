# XML Customer Metrics Report (`account_health_v1`)

Amazon's older XML customer metrics report, which "contains select, individual performance metrics data from the Seller Central dashboard": order defect, cancellation and late shipment metrics, policy statuses and buyer-seller contact response times, for each store you ask for ([Performance Reports](https://developer-docs.amazon/sp-api/docs/report-type-values-performance)).

## Where to find it in Seller Central

Not confirmed: no single screen matches this report. Amazon's report page says only "the Seller Central dashboard". Many of the metric names match those on the Account Health page ([Monitor your account health](https://sellercentral.amazon.co.uk/help/hub/reference/external/GX663LDD3TGU86GX)), which is **Account health > Manage account health** (`/performance/dashboard`; classic: **Performance > Account Health**). Checked in Seller Central UK on 28 Oct 2026. Its own menu has Customer Service Performance, Delivery Performance and a **Reports** drop-down with **Orders with Defects** and **Performance Over Time**.

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_V1_SELLER_PERFORMANCE_REPORT`, the marketplace ids of the stores you want, and no dates; then `getReport` and `getReportDocument`. Amazon says a report holds data "for all of the specified marketplaces, unless the report type indicates otherwise" ([createReport](https://developer-docs.amazon/sp-api/reference/createreport)); the performance page makes no exception for this report, so one file covers every store asked for. Amazon publishes no XML schema for it: there is none in [schemas/reports](https://github.com/amzn/selling-partner-api-models/tree/main/schemas/reports).
- **Report options:** none.
- **Role:** Selling Partner Insights ([role mappings for report types](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)).
- **How far back Amazon goes:** not stated in Amazon's docs. The report takes no date range. Amazon keeps no past copies, so history starts with the first copy you save.

## Fields

Checked against a real UK seller's file (Oct 2026): every element path, with each element's zero and non-zero values in every block. Paths below are under `sellerPerformanceReports/sellerPerformanceReport/`, one such block per store asked for. No element has attributes.

**No block says which store it is.** The file never names a marketplace. A block can sometimes be matched to its store through [`account_health`](account_health.md), which is labelled by store: a store can show `None` for valid tracking there, and a block with the same status is likely that store's. In the real file, the first block was the UK's; blocks for stores with no activity were identical, so they can't be told apart; and the blocks weren't in the order the marketplace ids were sent in.

Formats: rates are text with one decimal place and a percent sign (`N.N%`), not numbers. Statuses are `Good`, `Bad` or `None` (capitalised, unlike the JSON report's `GOOD`). Times are `YYYY-MM-DDTHH:MMZ`, without seconds. The file opens with a long XML comment listing Amazon's targets (order defect rate under 1%, pre-fulfilment cancel rate under 2.5%, late shipment rate under 4%, valid tracking over 95%).

| Field | Meaning | Format and traps |
|---|---|---|
| `performanceChecklist/<item>/status` | Pass or fail for each target. Items: `orderDefectRate`, `orderCancellationRate`, `lateShipmentRate`, `policyViolation`, `onTimeDelivery`, `validTrackingRate`, `contactResponseTime`, `returnDissatisfactionRate`, `customerServiceDissatisfactionRate`, `productAuthenticityStatus`, `productSafetyStatus`, `listingPolicyStatus`, `intellectualPropertyStatus`. | `Good`, `Bad` or `None`. |
| `orderDefects/orderDefectMetrics[]` | Order defect rate, twice per block with different periods: 60 days and 90 days, ending on different days before the download. | Neither entry is labelled AFN or MFN. |
| `…/orderDefectMetrics/timeFrame/start`, `…/timeFrame/end` | The period. | `YYYY-MM-DDTHH:MMZ`; the start is `T00:00Z` and the end `T23:59Z`. |
| `…/orderDefectMetrics/orderCount` | Orders in the period. | **Seller-fulfilled orders only.** In the real file, the 60-day count equalled `account_health`'s `orderDefectRate.mfn.orderCount` for the same days, and the 30-day `customerExperienceMetrics/orderCount` equalled its `lateShipmentRate.orderCount`. No count in this report holds FBA orders. `account_health` has two that do: `orderDefectRate.afn.orderCount` (Amazon's own 60-day FBA count, which didn't tie exactly to [`orders`](orders.md) for the same days, whether the days were taken as UTC, UK or US Pacific time) and `invoiceDefectRate.orderCount` (business orders only, 7 days). |
| `…/orderDefectMetrics/orderWithDefects/rate`, `/count` | Orders with a defect. | Rate as `N.N%`. |
| `…/orderDefectMetrics/negativeFeedbacks/rate`, `/count` | Defects from negative feedback. | Rate as `N.N%`. |
| `…/orderDefectMetrics/a_z_claims/rate`, `/count` | Defects from A-to-z claims. | Rate as `N.N%`. |
| `…/orderDefectMetrics/chargebacks/rate`, `/count` | Defects from chargebacks. | Rate as `N.N%`. |
| `customerExperience/customerExperienceMetrics[]` | Late shipment, pre-fulfilment cancellation and refund rates, twice per block (last 7 and last 30 days). | |
| `…/customerExperienceMetrics/timeFrame/start`, `/end`, `orderCount` | The period and its orders. | Seller-fulfilled orders only (see `orderCount` above). |
| `…/customerExperienceMetrics/lateShipment/rate`, `/count` | Late shipments. | Rate as `N.N%`. |
| `…/customerExperienceMetrics/preFulfillmentCancellation/rate`, `/count` | Cancellations before fulfilment. | Rate as `N.N%`. |
| `…/customerExperienceMetrics/refunds/rate`, `/count` | Refunds. | Rate as `N.N%`. |
| `trackingInformation/trackingMetrics[]` | Valid tracking and on-time delivery, twice per block (7 and 30 days, both ending on the same day, some days before the download). | Not in Amazon's list. |
| `…/trackingMetrics/validTracking/rate`, `/count` | Shipments with valid tracking. | Rate as `N.N%`. |
| `…/trackingMetrics/onTimeDelivery/rate`, `/count` | Shipments delivered on time. | Rate as `N.N%`. |
| `buyerSellerContactResponseTimeMetrics/responseTimeMetrics/timeFrame/start`, `/end` | The period (90 days). | Once per block. |
| `…/responseTimeMetrics/averageResponseTimeInHours` | Average reply time to buyer messages, in hours. | Can be `0` in every block, even an active store's, along with the three counts below: not measured, or no messages in the 90 days. `0` is not a real reply time. |
| `…/responseTimeMetrics/responseUnder24Hours` | Messages answered within 24 hours. | See `averageResponseTimeInHours`. |
| `…/responseTimeMetrics/responseTimeGreaterThan24Hours` | Messages answered after 24 hours. | See `averageResponseTimeInHours`. |
| `…/responseTimeMetrics/noResponseForContactsOlderThan24Hours` | Messages older than 24 hours with no reply. | See `averageResponseTimeInHours`. |
| `customerService/customerServiceMetrics[]` | Customer service dissatisfaction, three times per block (7, 30 and 90 days). | Not in Amazon's list. |
| `…/customerServiceMetrics/customerServiceDissatisfaction/rate`, `/count` | Dissatisfied contacts. | The count can be `0` in every block and period. |
| `returnDissatisfaction/returnDissatisfactionMetrics[]` | Return dissatisfaction, three times per block (7, 30 and 90 days). | Not in Amazon's list. |
| `…/returnDissatisfactionMetrics/returnOrderCount` | Returns in the period. | Can be `0` in every block and period, even an active store's; then the return rates below measure nothing. |
| `…/returnDissatisfactionMetrics/returnDissatisfaction/rate`, `/count` | Returns with a dissatisfied buyer. | Rate as `N.N%`. |
| `…/returnDissatisfactionMetrics/negativeReturnFeedback/rate`, `/count` | Negative feedback on returns. | Rate as `N.N%`. |
| `…/returnDissatisfactionMetrics/lateResponse/rate`, `/count` | Return requests answered late. | Rate as `N.N%`. |
| `…/returnDissatisfactionMetrics/invalidRejection/rate`, `/count` | Return requests wrongly rejected. | Rate as `N.N%`. |
| `productAuthenticityData`, `productSafetyData`, `listingPolicyData`, `intellectualPropertyData` | Each has `timeFrame/start`, `timeFrame/end` (180 days) and `defectCount`. | Not in Amazon's list. |

Meanings are read from the element names; Amazon lists the names and nothing more. Every name on Amazon's list is in the file. Not on the list: the `sellerPerformanceReports` and `sellerPerformanceReport` wrappers, `orderDefects`, `trackingInformation`, `trackingMetrics`, `validTracking`, `responseTimeMetrics`, the `customerService` and `returnDissatisfaction` groups with their own elements, the four `…Data` elements with `defectCount`, and the leaf names `start`, `end`, `rate`, `count` and `status`.

Source: [Performance Reports: XML Customer Metrics Report](https://developer-docs.amazon/sp-api/docs/report-type-values-performance).

## Personal data

None. No buyer or business personal details.

## Questions it answers

- Has Amazon started reporting buyer-message reply times? Look for a non-zero `averageResponseTimeInHours` or response counts; all `0` means no data, not instant replies.
- Have any returns or customer service contacts been measured (`returnOrderCount`, `customerServiceDissatisfaction/count`)? Only then do the dissatisfaction rates mean anything; [`account_health`](account_health.md) doesn't carry them.
- Are the product authenticity, product safety, listing policy and intellectual property statuses clear?
- How do order count, orders with defects, negative feedback, A-to-z claims and chargebacks compare with the same figures in [`account_health`](account_health.md)?

## Traps seen in real data

Seen in a real UK seller's file (Oct 2026):

- **Blocks aren't labelled by store.** The first was the UK's. Others can be matched through `account_health`'s statuses, such as a store that shows `None` for valid tracking. Blocks for stores with no activity were identical, and the blocks weren't in the order the marketplace ids were sent in.
- **Contact response time can be all zeros.** No other report here has it, but `averageResponseTimeInHours` and the three response counts can be `0` in every block, even an active store's: not measured, or no messages in the 90 days. Don't report it as a reply time.
- **Returns and customer service dissatisfaction can be all zeros too:** `returnOrderCount` and the dissatisfaction counts can be `0` in every block and period, so the rates built on them mean nothing.
- **The order counts are seller-fulfilled orders only.** For the same days they equal `account_health`'s `orderDefectRate.mfn.orderCount` and its `lateShipmentRate.orderCount`. For FBA order volume use [`orders`](orders.md). `account_health`'s `orderDefectRate.afn.orderCount` is Amazon's own 60-day FBA count and doesn't tie exactly to `orders` for the same days, whichever time zone you use. Its `invoiceDefectRate.orderCount` counts FBA orders too, but only business ones.
- **Rates are text** (`N.N%`), and statuses are `Good`/`Bad`/`None`, not the JSON report's capitals.
