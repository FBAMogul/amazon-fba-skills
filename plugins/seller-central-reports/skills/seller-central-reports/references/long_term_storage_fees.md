# Aged Inventory Surcharge / Long-Term Storage Fees (`long_term_storage_fees`)

Each aged inventory surcharge (long-term storage fee) Amazon charged, per SKU, country and age tier, one finished calendar month per request: the charge data for long-term storage ([SP-API FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). In the UK and EU Amazon now calls this charge the aged inventory surcharge: it applies to units stored 241 days or more, assessed on the 15th of every month ([FBA aged inventory surcharge](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200684750)). Each row has the quantity charged, unit volume, age tier and amount ([Aged Inventory Surcharge report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200725880)). A row is one SKU in one country in one age tier, not one SKU: add up a SKU's rows for its charge (see Traps).

## Where to find it in Seller Central

Not in the Supply chain reports list when checked in Seller Central UK on 28 Oct 2026. That list (**Supply chain > Supply chain reports**, the Amazon Fulfilment Reports page; classic: **Reports > Fulfilment by Amazon**; [Amazon Fulfilment reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201104750)) had no aged inventory surcharge or long-term storage fee report: its **Payments** section held Fee Preview, Inventory storage overage fees, Monthly Storage Fees, Returns Processing Fee and Reimbursements. The UK help still calls it the **Aged Inventory Surcharge report** ([G200725880](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200725880)) or the **Long-Term Storage Fee report** ([GJQNPA23YWVA4SBD](https://sellercentral.amazon.co.uk/help/hub/reference/external/GJQNPA23YWVA4SBD)), and both link to `/reportcentral/LONGTERM_STORAGE_FEE_CHARGES`; whether that address still opens the report is not confirmed.

Two other Seller Central reports cover aged stock. The FBA Inventory report (**Supply chain > Manage Amazon inventory**, then **Reports > Inventory report**; the [inventory_planning](inventory_planning.md) data) shows "aged inventory (including applicable fees)", in Amazon's words. The SKU Economics report (**Finance > Profit analytics > Selling economics**, **SKU Economics report** tab; see [seller_economics](seller_economics.md)) has aged inventory surcharge columns per SKU.

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_FBA_FULFILLMENT_LONGTERM_STORAGE_FEE_CHARGES_DATA`, every store's marketplace id, and `dataStartTime`/`dataEndTime` set to one finished calendar month (the 1st at 00:00:00Z to the last day at 23:59:59Z), with no report options; then poll `getReport` until it's done and fetch the file with `getReportDocument`. Amazon says the span from `dataStartTime` to `dataEndTime` "should be equal to one month" ([SP-API FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)).
- **Role:** Amazon Fulfillment or Pricing ([role mappings](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)).
- **How far back Amazon goes:** not stated in Amazon's docs. Seen in a real UK seller's account (Oct 2026): Amazon returned months up to 12 back, cancelled the month 13 back, and returned FATAL for older months, though `transactions` can show the surcharge in those months. So the report just doesn't reach past 12 months. For older months use `transactions` (see Traps).

## Columns

Checked against a real UK seller's files in Oct 2026: 14 columns, in this order, the same in every month. Tab-separated text, header row first.

| Column | Meaning | Format and traps |
|---|---|---|
| `snapshot-date` | Date the surcharge was charged. | ISO 8601 date-time with offset, e.g. `…T00:01:00+00:00`. Always the 15th of the month asked for, at `00:00` to `00:02` UTC. |
| `sku` | Your SKU. | |
| `fnsku` | Amazon's fulfilment network SKU. | |
| `asin` | Amazon's product identifier. | |
| `product-name` | Product title. | |
| `condition` | Condition of the product. | A condition such as `New`, or `No Listing`: aged stock with no live offer still gets charged. |
| `per-unit-volume` | Volume per unit for which the surcharge was charged. | In `volume-unit`. Small values (seen in cubic metres) can come in scientific notation, in the form `n.nE-4`: read them as numbers. |
| `currency` | Not defined on the UK help page. | A currency code such as `GBP` or `EUR`. |
| `volume-unit` | Unit of the volume value. | `cubic_feet` for `GB`, `cubic_meters` for EU countries. |
| `country` | Country the surcharge is charged under, not always where the units are stored. | Two letters, such as `GB` or `DE`. **A code can cover stock held in other countries' centres** (see Traps). There's no centre column. |
| `qty-charged` | Units the surcharge was assessed on. | Whole number. |
| `amount-charged` | Surcharge charged for the SKU. | In `currency`, before VAT. Hyphenated, not the help page's `amount_charged`. Sum this for totals; never recompute it from `rate-surcharge`, which is rounded. |
| `surcharge-age-tier` | Age tier charged. | A range of days, such as `241-270`. Up to January 2026 the top tier is `365+`; from February 2026 it splits into `366-455` and `456+`. |
| `rate-surcharge` | Per-unit rate: `per-unit-volume` x the tier's surcharge rate, or a flat per-unit minimum where that is higher. | In `currency`. Rounded, so `rate-surcharge` x `qty-charged` can differ from `amount-charged`. The flat minimum shows only in the `366-455` and `456+` tiers: there `rate-surcharge` can be above volume x rate even allowing for rounding, and one amount per tier across SKUs of different volumes. |

Every column Amazon lists is in the file.

## Personal data

None: no buyer or business details. Some sellers put their cost price in the SKU, so treat SKUs as private. Rule: strip personal data before analysis and never copy it into anything shared.

## Questions it answers

- Which SKUs were charged the aged inventory surcharge each month, and how much in total (sum `amount-charged`, one `currency` at a time)?
- Which age tiers is old stock sitting in, and is it moving into dearer tiers month on month?
- Which countries' fulfilment centres hold the aged stock? Not from `country` alone: a code can cover centres in other countries. Join each row to `storage_fees` on `fnsku`, month and country (`country` to its `country_code`), and go by its `fulfillment_center`.
- Which bought lines age badly enough that removal or liquidation would have been cheaper?

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Only 12 months back, even where older months were charged.** The month 13 back comes back cancelled and older ones FATAL, even for months where `transactions` has `FBALongTermStorageBilling` lines. A cancelled month doesn't mean nothing was charged.
- **Older months: totals from `transactions`.** Take the lines with `description` `FBALongTermStorageBilling` (`transactionType` `ServiceFee`), one per country's store in each month charged, and add up the item-level `Base`: `AmazonFees > FBALongTermStorageFee > Base` up to April 2026, `FBAFees > LongTermStorageFee > FBALongTermStorageFee > Base` from May 2026. `Base` leaves out VAT, like `amount-charged`, but it's negative (a charge), while `amount-charged` is positive (it can be `0`): turn the sign round before comparing or adding. The items carry no SKU or ASIN, so this gives a total per country, not per SKU. Where both exist, a country-month's `amount-charged` sum matches minus its `Base`, usually exactly and otherwise within a fraction of a percent.
- **Several rows per SKU.** A SKU can have units in two or three age tiers, or be charged in two countries, in the same month. Month, `sku`, `country` and `surcharge-age-tier` together are unique. Add up the rows per SKU, one `currency` at a time.
- **A country code isn't always where the stock is.** A row's `country` is the store the surcharge is charged under, and that store's stock can be held in other countries' centres (see the `country_code` column in [storage_fees](storage_fees.md)). There's no centre column here, so join each row's FNSKU, month and country to `storage_fees` for the centres; where that shows more than one centre country, the surcharge can't be split between them.
- **`No Listing` condition:** aged stock with no live offer still gets charged.
- **Mixed units and currencies:** cubic feet and GBP for `GB` rows, cubic metres and EUR for eurozone stores' rows.
- Lines end in CRLF, but the last line of the file has no line ending at all, so `wc -l` counts one row short. The text is UTF-8.

Sources: [SP-API FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba) (column names) and [Aged Inventory Surcharge report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200725880) (meanings).
