# Seller Performance Report: Account Health (`account_health`)

Amazon's Seller Performance Report, which "contains the individual performance metrics data from the Seller Central Account Health dashboard": account status, performance metrics against Amazon's targets, the Account Health Rating and policy-violation counts, for each store you ask for ([Performance Reports](https://developer-docs.amazon/sp-api/docs/report-type-values-performance)).

## Where to find it in Seller Central

**Account health > Manage account health** (`/performance/dashboard`). Classic: **Performance > Account Health**. Checked in Seller Central UK on 28 Oct 2026. The page's own menu has Account Health, Customer Service Performance, Product Policy Compliance, Delivery Performance, a **Reports** drop-down (**Orders with Defects**, **Performance Over Time**), Eligibilities and Voice of the Customer; the tabs above it are Manage account health, Performance notifications and Manage compliance. No download button was seen on Orders with Defects or Performance notifications. Amazon's help page says: "To view your account health, in Seller Central go to Account Health" ([Monitor your account health](https://sellercentral.amazon.co.uk/help/hub/reference/external/GX663LDD3TGU86GX)).

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_V2_SELLER_PERFORMANCE_REPORT`, the marketplace ids of the stores you want, and no dates; then `getReport` and `getReportDocument`. One file covers every store asked for:
  - Amazon says a report holds data "for all of the specified marketplaces, unless the report type indicates otherwise" ([createReport](https://developer-docs.amazon/sp-api/reference/createreport)), and the performance page makes no exception for this report.
  - The schema has one `accountStatuses` entry and one `performanceMetrics` entry for each requested marketplace ([schema](https://github.com/amzn/selling-partner-api-models/blob/main/schemas/reports/accountHealthReport-2020-11-18.json)).
- **Report options:** none.
- **Role:** Selling Partner Insights ([role mappings for report types](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)).
- **How far back Amazon goes:** not stated in Amazon's docs. The report takes no date range; each metric carries its own `reportingDateRange`. Amazon keeps no past copies, so history starts with the first copy you save.

## Fields

Checked against a real UK seller's file (Oct 2026): every field path. The file is JSON: one object with just two keys, `accountStatuses` and `performanceMetrics`.

How to read the table:

- **Field names:** the file matches Amazon's report page, including the four fields its older GitHub schema lacks (`lateShipmentRateList`, `accountHealthRating.ahrScore`, `customerProductReviewsPolicyViolations`, `policyViolationWarnings`).
- **Common fields:** most metric objects share these fields, described once in the first rows of the table:
  - `reportingDateRange.reportingDateFrom` and `reportingDateRange.reportingDateTo`
  - `status`
  - `targetValue`
  - `targetCondition`
- **Listing-level shape:** the "listing-level violation" objects hold `reportingDateRange`, `defectsCount`, `status`, `targetValue` and `targetCondition`.

| Field | Meaning | Format and traps |
|---|---|---|
| `<metric>.reportingDateRange.reportingDateFrom` | Start of the period the metric covers. | `YYYY-MM-DDTHH:MM:SSZ`, UTC. Each metric has its own period. |
| `<metric>.reportingDateRange.reportingDateTo` | End of that period. | UTC. Most metrics run to the day after the download; order defect rate, on-time delivery, unit on-time delivery, valid tracking, invoice defect rate and pre-fulfilment cancellation end earlier, by up to about two weeks. |
| `<metric>.status` | Status of the metric for the period. | `GOOD`, `BAD` or `NONE`. |
| `<metric>.targetValue` | Target value to stay in good status. | A fraction: `0.04` is 4%. |
| `<metric>.targetCondition` | How the value is compared with the target. | `GREATER_THAN`, `LESS_THAN` or `EQUALS`. |
| `<metric>.rate` | The metric's rate for the period. | Scale not confirmed: the schema says "rate in percentage" but `targetValue` is a fraction, and the real values checked don't settle it. |
| `accountStatuses[].marketplaceId` | The store. | One entry per store asked for. |
| `accountStatuses[].status` | Account status in that store. | `NORMAL`, `AT_RISK` or `DEACTIVATED`. |
| `performanceMetrics[].marketplaceId` | The store. | One entry per store asked for. |
| `performanceMetrics[].lateShipmentRate` | Late shipment rate: common fields plus `orderCount`, `lateShipmentCount` and `rate`. | 30-day period. |
| `performanceMetrics[].lateShipmentRateList[]` | The same fields as `lateShipmentRate`, once per period. | Not in the schema. Two periods per store: the last 10 days and the last 30 days (the second matches `lateShipmentRate`). |
| `performanceMetrics[].invoiceDefectRate` | Invoice defect rate: common fields plus `orderCount` and `rate`. | 7-day period. |
| `performanceMetrics[].invoiceDefectRate.invoiceDefect.count` / `.status` | Total invoice defects, from missing and late invoices. | |
| `performanceMetrics[].invoiceDefectRate.missingInvoice.count` / `.status` | Missing invoices in the period. | |
| `performanceMetrics[].invoiceDefectRate.lateInvoice.count` / `.status` | Late invoices in the period. | |
| `performanceMetrics[].orderDefectRate.afn` | Order defect rate for Amazon-fulfilled orders (fields below). | 60-day period. |
| `performanceMetrics[].orderDefectRate.mfn` | Order defect rate for seller-fulfilled orders (same fields). | |
| `….orderDefectRate.<afn or mfn>.orderCount` | Orders in the period. | Zero in stores with no orders. |
| `….orderDefectRate.<afn or mfn>.orderWithDefects.count` / `.status` | Orders with defects. | |
| `….orderDefectRate.<afn or mfn>.claims.count` / `.status` | Order defects due to claims. | |
| `….orderDefectRate.<afn or mfn>.chargebacks.count` / `.status` | Order defects due to chargebacks. | |
| `….orderDefectRate.<afn or mfn>.negativeFeedback.count` / `.status` | Order defects due to negative feedback. | |
| `….orderDefectRate.<afn or mfn>.fulfillmentType` | Fulfilment type of this block. | `AFN` or `MFN`. |
| `….orderDefectRate.<afn or mfn>.rate`, `.status`, `.targetValue`, `.targetCondition`, `.reportingDateRange` | Common fields. | `targetValue` `0.01` (1%). |
| `performanceMetrics[].onTimeDeliveryRate` | On-time delivery rate: common fields plus `shipmentCountWithValidTracking`, `onTimeDeliveryCount` and `rate`. | `targetValue` `0.97`. |
| `performanceMetrics[].unitOnTimeDeliveryRate` | On-time delivery at unit level: common fields plus `unitOnTimeDeliveryCount`, `totalUnitCount` and `rate`. | Present for the UK and EU stores, though the schema says US only. `targetValue` is `0.9` in the UK, DE, FR, IT and ES, and `0.95` in NL, SE, PL, BE and IE. |
| `performanceMetrics[].validTrackingRate` | Valid tracking rate: common fields plus `shipmentCount`, `validTrackingCount` and `rate`. | `targetValue` `0.95`. |
| `performanceMetrics[].preFulfillmentCancellationRate` | Pre-fulfilment cancellation rate: common fields plus `orderCount`, `cancellationCount` and `rate`. | `targetValue` `0.025`. 7-day period. |
| `performanceMetrics[].warningStates` | Seller warning states. | Deprecated; an empty array. |
| `performanceMetrics[].accountHealthRating.ahrStatus` | Overall Account Health Rating status for the period. | The schema says it "could be GREAT or WARNED" (under the name `accountHealthRatingStatus`). |
| `performanceMetrics[].accountHealthRating.ahrScore` | The Account Health Rating score, 0 to 1,000 ([Account Health Rating](https://sellercentral.amazon.co.uk/help/hub/reference/external/GR786P4BPEVKTBAG)). | Exactly `200.0` in stores with no orders, which looks like a default. |
| `performanceMetrics[].accountHealthRating.reportingDateRange` | Period of the rating. | 181 days: one day longer than the listing-level objects' 180. |
| `performanceMetrics[].listingPolicyViolations`, `productAuthenticityCustomerComplaints`, `productConditionCustomerComplaints`, `productSafetyCustomerComplaints`, `receivedIntellectualPropertyComplaints`, `restrictedProductPolicyViolations`, `suspectedIntellectualPropertyViolations`, `foodAndProductSafetyIssues`, `customerProductReviewsPolicyViolations`, `otherPolicyViolations`, `documentRequests` | One object each, in the listing-level shape. | 180-day period. `customerProductReviewsPolicyViolations` isn't in the schema but is in the file. |
| `….<listing-level>.defectsCount` | Number of defects in the period for that store. | `targetValue` `0`, `targetCondition` `EQUALS`. |
| `performanceMetrics[].policyViolationWarnings.warningsCount` | Count of policy violation warnings. | Not in the schema. |
| `performanceMetrics[].policyViolationWarnings.reportingDateRange` | Period of those warnings. | 181 days. Not in the schema. |

Every field on Amazon's report page is in the file.

Source: [Performance Reports: Seller Performance Report](https://developer-docs.amazon/sp-api/docs/report-type-values-performance) and the schema [accountHealthReport-2020-11-18.json](https://github.com/amzn/selling-partner-api-models/blob/main/schemas/reports/accountHealthReport-2020-11-18.json) (meanings and allowed values come from the schema).

## Personal data

None. No buyer or business personal details.

## Questions it answers

- Is the account `NORMAL` in every store, or is any store `AT_RISK` or `DEACTIVATED`?
- What are the Account Health Rating status and score per store, and how have they moved since earlier copies?
- Are there any authenticity, condition, intellectual property or restricted-product complaints, and in which store?
- Is the order defect rate for FBA orders and for seller-fulfilled orders under target?
- Is the invoice defect rate for Amazon Business orders under target?

## Traps seen in real data

Seen in a real UK seller's file (Oct 2026):

- **Stores with no orders still get a full block,** with zero counts and an `ahrScore` of exactly `200.0`, which looks like a default for a store with no orders. Don't read it as a real score.
- **A `BAD` status doesn't say why.** The file gives counts only; it doesn't name the ASIN or notice behind a `BAD` status. The Account Health page in Seller Central does. Amazon's Listings Items API may show some of them as issues on a listing (see [`not-in-the-api.md`](not-in-the-api.md)).
- **The rate scale is unconfirmed.** The real `rate` values don't show whether rates are fractions (like `targetValue`) or percentages.
- **`unitOnTimeDeliveryRate` is present** for UK and EU stores, not just the US as the schema says.
- **Periods differ by metric:** 7, 30, 60, 180 or 181 days, and some end up to about two weeks before the download. Compare each metric with its own `reportingDateRange`.
