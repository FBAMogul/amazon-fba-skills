# Seller feedback report (`seller_feedback`)

Neutral and negative seller feedback (1 to 3 stars) from buyers, with the comment, any reply and the order id: Amazon's Flat File Feedback Report, "negative and neutral feedback (one to three stars) from buyers who rated the seller's performance" ([Amazon](https://developer-docs.amazon/sp-api/docs/report-type-values-performance)), the data shown in Feedback Manager's feedback table ([UK help](https://sellercentral.amazon.co.uk/help/hub/reference/external/G202125660)); positive feedback (4 and 5 stars) is not in it.

## Where to find it in Seller Central

**Products > Product reports**, then the **Feedback report** tab (`/feedback-manager/index.html#/report`, part of Feedback Manager). Feedback Manager itself is **Products > Analyse reviews > Feedback Manager** (`/feedback-manager/index.html`). Classic: **Performance > Feedback Manager**. Checked in Seller Central UK on 28 Oct 2026. Amazon's help names the download step **Download feedback reports** ([UK help](https://sellercentral.amazon.co.uk/help/hub/reference/external/G202125660)).

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_SELLER_FEEDBACK_DATA`, the marketplace ids of the stores you want, and `dataStartTime`/`dataEndTime` set to the period; then `getReport` until the report is done, and `getReportDocument` (unzip it if Amazon says GZIP). No report options.
- **Role:** Amazon Fulfillment or Selling Partner Insights ([Amazon](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)).
- **How far back Amazon goes:** not stated in Amazon's SP-API docs. In Seller Central a report covers "a selected period of up to one year" ([UK help](https://sellercentral.amazon.co.uk/help/hub/reference/external/G202125660)). Buyers have 90 days to leave feedback and 60 days to remove it ([UK help](https://sellercentral.amazon.co.uk/help/hub/reference/external/G761)).
- **Empty periods:** Amazon cancels a report automatically when there is no data to return ([Amazon](https://developer-docs.amazon/sp-api/reference/getreport)). A period with no neutral or negative feedback may therefore come back `CANCELLED` with no document.

## Columns

Tab-separated text, header row first. Not checked against a real file; the table is from Amazon's docs.

| Column | Meaning | Format and traps |
|---|---|---|
| `Date` | The date the feedback was received. | Format not documented. |
| `Rating` | The buyer's star rating. | 1 to 3 only (3 neutral, 1 and 2 negative). |
| `Comments` | The buyer's feedback comment. | Free text from a buyer: never copy it anywhere public. |
| `Your Response` | Not defined in Amazon's docs. Feedback Manager lets the seller post a public reply. | |
| `Order ID` | The order the feedback is for. | |
| `Rater Email` | The rater's email. | Deprecated: always `no-reply@amazon.com`. |

There is no store column; join `Order ID` to `orders` to get `sales-channel`.

Source: [Performance Reports: Flat File Feedback Report](https://developer-docs.amazon/sp-api/docs/report-type-values-performance); `Date` and `Rating` from [Which tools are used to manage seller feedback? (UK help)](https://sellercentral.amazon.co.uk/help/hub/reference/external/GRUKMKGABC9J3VLA) and [About Feedback Manager (UK help)](https://sellercentral.amazon.co.uk/help/hub/reference/external/G761).

## Personal data

`Comments` is the buyer's own free text and can hold personal details. `Rater Email` is deprecated and always `no-reply@amazon.com`. Strip `Comments` before analysis and never copy it into anything shared. So Claude doesn't read or summarise what buyers wrote; the seller can read the comments themselves in Seller Central. Strip `Your Response` too: a public reply is free text and can quote the buyer or name them. To answer which feedback has no reply yet, have the stripping script first add a yes/no column saying whether `Your Response` was filled.

## Questions it answers

- Which orders got neutral or negative feedback, and with what `Rating`?
- Which ASINs or stores attract most negative feedback? (Join to `orders`.)
- Which negative feedback has no reply yet?
- Is negative feedback rising month by month?

## Traps seen in real data

Seen in a real UK seller's account (Oct 2026):

- **Cancelled can just mean no data.** A period with no neutral or negative feedback comes back `CANCELLED`, with no file, not an empty file. Per Amazon's Reports FAQ, a report is cancelled by a manual cancel, when there's no data, or when the request falls inside the report's once-per-period limit.
- **Neutral feedback isn't counted anywhere else.** For feedback counts use `sales_and_traffic` (per day: `feedbackReceived`, `negativeFeedbackReceived`) or `account_health` (negative feedback, 60 days per store), but neither counts 3-star feedback on its own, so for neutral feedback only this report's cancellation says there was none, unless your own code cancelled the request or it fell inside the once-per-period limit. When the report is cancelled, the comments themselves are only in Seller Central's Feedback Manager.
- **Ask again later for recent months.** Buyers can leave feedback for 90 days, so a month that was empty when first asked for can fill later. Feedback a buyer removes stays in any copy you downloaded before.
