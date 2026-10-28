# Suppressed listings report (`suppressed_listings`)

Listings Amazon has suppressed in one store (hidden from search and browse, for example for a missing main image, missing price-per-unit data or a title that breaks the title rules), with the reason for each and instructions for removing the suppression ([SP-API inventory reports](https://developer-docs.amazon/sp-api/docs/report-type-values-inventory), [Suppressed listings](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200898440)).

## Where to find it in Seller Central

Amazon's help puts the suppressed listings themselves at **Inventory > Manage all inventory > Activate listings** ([Suppressed listings](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200898440)). Classic Seller Central still has **Inventory > Manage All Inventory**; the new menu's nearest entry is **Products > Manage products**. Neither was opened for suppressed listings, so the rest of the route is not confirmed.

That help page also mentions a downloadable "Listing Quality and Suppressed Listings report"; whether it is this report: not confirmed. Neither it nor any suppressed listings report was in the **Products > Product reports** request list (classic: **Reports > Inventory Reports**) or the **Supply chain > Supply chain reports** list when checked in Seller Central UK on 28 Oct 2026.

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_MERCHANTS_LISTINGS_FYP_REPORT` and no `dataStartTime`/`dataEndTime`; then `getReport` and `getReportDocument`. Amazon accepts only one store per report: given several marketplace ids, it uses only the first ([SP-API inventory reports](https://developer-docs.amazon/sp-api/docs/report-type-values-inventory)). So ask store by store, each with that store's marketplace id only ([Store Identifiers](https://developer-docs.amazon/sp-api/docs/store-identifiers)).
- **Report options:** Amazon accepts `preferredReportDocumentLocale` (the language of the column headers) ([SP-API inventory reports](https://developer-docs.amazon/sp-api/docs/report-type-values-inventory)).
- **Role:** Product Listing ([role mappings](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)).
- **How far back Amazon goes:** not stated in Amazon's docs. Each file is the suppressions as they stood when asked for. Amazon can't make one for a past date, so history starts with the first copy you save. A copy your app requested in the last 90 days can still be fetched (`getReports`, then `getReportDocument`).

## Columns

Checked against a real UK seller's header rows (Oct 2026), one file per store: the same 8 columns, in English, in each. The file is tab-separated text with a header row first. It starts with a UTF-8 byte order mark, and lines end in LF. The values and the date format are not yet confirmed.

| Column | Meaning | Format and traps |
|---|---|---|
| `Status` | Not defined in Amazon's docs. | |
| `Reason` | Not defined in Amazon's docs. | |
| `SKU` | Your SKU. | |
| `ASIN` | Amazon's product identifier. | |
| `Product name` | The product title. | |
| `Condition` | Not defined in Amazon's docs. | |
| `Status Change Date` | Not defined in Amazon's docs. | Date format not confirmed. |
| `Issue Description` | Not defined in Amazon's docs. | |

Every column Amazon lists is in the file, with the same capitals.

Source: [SP-API inventory reports](https://developer-docs.amazon/sp-api/docs/report-type-values-inventory) (column names only).

## Personal data

None. No buyer or business personal details.

## Questions it answers

- Which SKUs are suppressed in each store right now, and why?
- How long has each SKU been suppressed (`Status Change Date`), and so hidden from search?
- Which suppressed SKUs still hold FBA stock? Join on SKU with the inventory reports; that stock is earning nothing and still paying storage.
- Which suppression reasons keep coming back across the copies you save over time?

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **A store with nothing suppressed** comes back as a header row only.
- **Meanings, values and the date format are unconfirmed.** Check them against a file with data rows.
- Each file starts with a UTF-8 byte order mark; lines end in LF.
