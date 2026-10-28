# Seller-fulfilled returns report (`fbm_returns`)

Returns of seller-fulfilled (FBM) orders, by return request date: Amazon's Flat File Returns Report by Return Date, with return request date, RMA ID, label details, ASIN and return reason code, covering only seller-fulfilled order returns, not FBA returns or returnless refunds ([Amazon](https://developer-docs.amazon/sp-api/docs/report-type-values-returns)); FBA returns are in `fba_returns`.

## Where to find it in Seller Central

**Orders > Order reports**, then the **Seller fulfilled return reports** tab (`/returns/report`). Classic: **Reports > Return Reports** (the same address). Checked in Seller Central UK on 28 Oct 2026. Amazon's help gives a route from the returns page: **Orders > Manage returns > View reports**, then pick the return type and time frame ([UK help](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200708210)).

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_FLAT_FILE_RETURNS_DATA_BY_RETURN_DATE`, the marketplace ids of the stores you want, and `dataStartTime`/`dataEndTime` set to the period; then `getReport` until the report is done, and `getReportDocument` (unzip it if Amazon says GZIP). No report options.
- **Role:** Inventory and Order Tracking, or Direct to Consumer Shipping (restricted) ([Amazon](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)).
- **How far back Amazon goes:** not stated in Amazon's docs. Each request covers up to 60 days ([Amazon](https://developer-docs.amazon/sp-api/docs/report-type-values-returns)).
- **Empty periods:** Amazon's docs say it cancels a report when there is no data to return ([Amazon](https://developer-docs.amazon/sp-api/reference/getreport)), but for this report it sends a header-only file instead (see Traps).

## Columns

Tab-separated text, header row first. Columns as seen in the header row of a real UK seller's files (Oct 2026), in this order. The formats of the values aren't confirmed.

| Column | Meaning | Format and traps |
|---|---|---|
| `Order ID` | Not confirmed; by name, the Amazon order id. | |
| `Order date` | Not confirmed. | |
| `Return request date` | The date the return was requested. | |
| `Return request status` | Not confirmed. | |
| `Amazon RMA ID` | Amazon's return merchandise authorisation (RMA) ID. | |
| `Merchant RMA ID` | Not confirmed. | |
| `Label type` | Return label detail: the label type. | |
| `Label cost` | Return label detail: the label cost. | |
| `Currency code` | Not confirmed. | |
| `Return carrier` | Return label detail: the carrier. | |
| `Tracking ID` | Return label detail: the tracking ID. | |
| `Label to be paid by` | Return label detail: who pays for the label. | |
| `A-to-Z Claim` | Not confirmed. | |
| `Is prime` | Not confirmed. | |
| `ASIN` | The ASIN returned. | |
| `Merchant SKU` | Not confirmed. | |
| `Item Name` | Not confirmed. | |
| `Return quantity` | Not confirmed. | |
| `Return Reason` | The return reason code. | |
| `In policy` | Not confirmed. | |
| `Return type` | Not confirmed. | |
| `Resolution` | Not confirmed. | |
| `Invoice number` | Not confirmed. | |
| `Return delivery date` | Not confirmed. | |
| `Order Amount` | Not confirmed. | |
| `Order quantity` | Not confirmed. | |
| `SafeT Action reason` | Not confirmed. | |
| `SafeT claim id` | Not confirmed. | |
| `SafeT claim state` | Not confirmed. | |
| `SafeT claim creation time` | Not confirmed. | |
| `SafeT claim reimbursement amount` | Not confirmed. | |
| `Refunded Amount` | Not confirmed. | |
| `Category` | Not in Amazon's docs. | Sits between `Refunded Amount` and `VAT`. |
| `VAT` | Not in Amazon's docs. | |
| `Order Item ID` | Not confirmed. | Last column. |

Amazon lists the column names (all but `Category` and `VAT`) but defines none of them. The meanings given come from the report's description ("return request date, RMA ID, label details, ASIN, and return reason code"). There is no store column; join `Order ID` to `orders` to get `sales-channel`.

Source: [Returns Reports: Flat File Returns Report by Return Date](https://developer-docs.amazon/sp-api/docs/report-type-values-returns).

## Personal data

None by column name: there is no buyer name, address, email or phone column. The values themselves aren't checked.

## Questions it answers

- Were there any seller-fulfilled returns in a period?
- Which FBM orders were returned, and why?
- Were any SAFE-T claims raised or reimbursed on FBM returns?

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Header-only, not cancelled.** For a period with no FBM returns, Amazon doesn't cancel the report, as its docs suggest; it builds a file with the header row and nothing else. Check whether a file has any rows below the header, not whether the request was cancelled.
- This report's lines don't end in CRLF, unlike the FBA reports.
