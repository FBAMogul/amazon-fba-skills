# Monthly Storage Fees (`storage_fees`)

Amazon's estimated monthly storage fee for each ASIN of your stock in each fulfilment centre, one calendar month per request ([SP-API FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). Amazon charges storage per calendar month on the daily average volume, usually between the 7th and 15th of the following month, and the fee is a base rate plus, for some sellers, a storage utilisation surcharge ([Monthly storage fees](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200612770)).

## Where to find it in Seller Central

**Supply chain > Supply chain reports** (the Amazon Fulfilment Reports page; [Amazon Fulfilment reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201104750)), then **Payments > Monthly Storage Fees** in the left-hand list (`/reportcentral/STORAGE_FEE_CHARGES/1`, the address the UK help links: [Monthly Storage Fees report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G202086720)). **Inventory storage overage fees**, in the same section, is a different report. Classic: **Reports > Fulfilment by Amazon**. Checked in Seller Central UK on 28 Oct 2026.

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_FBA_STORAGE_FEE_CHARGES_DATA`, every store's marketplace id, and `dataStartTime`/`dataEndTime` set to one calendar month (the 1st at 00:00:00Z to the last day at 23:59:59Z), with no report options; then poll `getReport` until it's done and fetch the file with `getReportDocument`.
- **Role:** Amazon Fulfillment or Pricing ([role mappings](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)).
- **How far back Amazon goes:** not stated in Amazon's docs. Amazon also doesn't say how `dataStartTime`/`dataEndTime` select rows (by `month_of_charge` or otherwise) ([SP-API FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)).
- **Throttling (not documented by Amazon):** seen in a real UK seller's account in Oct 2026, Amazon refused `createReport` for this report type with HTTP 429 after about 6 requests in a day, while still accepting other report types. When the count resets isn't clear. Ask for a few months at a time, and for more, ask again a day after the last accepted request.

## Columns

Checked against a real UK seller's files in Oct 2026: 34 columns, in this order, the same in every month. Tab-separated text, header row first. The names use underscores, as on the SP-API page, not the help page's hyphens.

| Column | Meaning | Format and traps |
|---|---|---|
| `asin` | Amazon's product identifier. | |
| `fnsku` | Amazon's fulfilment network SKU. | No SKU column; join on `fnsku`. |
| `product_name` | Description of the ASIN on the detail page. | |
| `fulfillment_center` | Fulfilment centre where the ASIN is stored. | **Wrapped in single quotes:** strip them. Not one row per ASIN per centre: an ASIN, or an FNSKU, can have several rows in one centre. |
| `country_code` | Country the stock is billed under, not always the centre's country. | Two-letter code, e.g. `GB`. **A store's code can cover centres in other countries:** under Amazon's [Central Europe Programme](https://sell.amazon.de/versand-durch-amazon/central-europe-program), Amazon.de stock can be stored in Germany, Poland or the Czech Republic. For the centre's country, go by `fulfillment_center`. |
| `longest_side` | Longest side of one unit. | Decimal. |
| `median_side` | Side that is neither longest nor shortest. | Decimal. |
| `shortest_side` | Shortest side of one unit. | Decimal. |
| `measurement_units` | Unit for the three sides. | `centimeters`. |
| `weight` | Weight used for the size tier: unit weight or dimensional weight. | Decimal. |
| `weight_units` | Unit for `weight`. | `kilograms` (grams in `fba_fee_preview`). |
| `item_volume` | Volume of one unit (longest x median x shortest side). | In `volume_units`. Can come in scientific notation (see Traps). |
| `volume_units` | Unit for the volume. | `cubic feet` for `GB` rows, `cubic meters` for EU rows. |
| `product_size_tier` | Size classification used for fees. | `Standard-Size` or `Oversize`. |
| `average_quantity_on_hand` | Daily average units in fulfilment centres: the month's inventory divided by its days. | One or two decimals. |
| `average_quantity_pending_removal` | Daily average units with a removal requested. | Decimal. |
| `estimated_total_item_volume` | (Average on hand minus average pending removal) x unit volume. | `0` where an incentive covers the whole fee, though stock was on hand. |
| `month_of_charge` | Month the fee estimate applies to. | `YYYY-MM`. Always the month asked for. |
| `storage_utilization_ratio` | Storage utilisation ratio for the size tier on the last day of the month. | A number of weeks, or text such as `>52` or `--`, so not always a number. |
| `storage_utilization_ratio_units` | Unit for the ratio. | `weeks` or `--`. |
| `base_rate` | Base monthly storage fee rate that applies to this ASIN. | Per unit of volume, in `currency`. Amazon's rate card, so many rows share a value. |
| `utilization_surcharge_rate` | Storage utilisation surcharge rate that applies, set by size tier and ratio. | Decimal. |
| `avg_qty_for_sus` | Not defined in Amazon's docs. | By name, the average quantity the surcharge applies to. |
| `est_vol_for_sus` | Not defined in Amazon's docs. | By name, the volume the surcharge applies to. |
| `est_base_msf` | Not defined in Amazon's docs. | By name, the estimated base monthly storage fee. Can be `0`, with `est_sus`, on rows that carry a fee, so don't rebuild the fee from it. |
| `est_sus` | Not defined in Amazon's docs. | By name, the estimated utilisation surcharge. |
| `currency` | Currency of the storage rate amount. | `GBP` on `GB` rows, `EUR` on rows for euro-country stores. |
| `estimated_monthly_storage_fee` | Monthly storage fee that applies to this ASIN. | In `currency`. **After any incentive:** `total_incentive_fee_amount` is already taken off. Sum every row for a month's fee; the sum equals the storage billed, before VAT, in `transactions` and `settlements`. |
| `category` | Item category used to calculate storage fees. | Text, e.g. `Others`. On the UK help page, not the SP-API list. |
| `eligible_for_inventory_discount` | Not defined in Amazon's docs. | By name, a yes/no flag (`N` for no). |
| `qualifies_for_inventory_discount` | Not defined in Amazon's docs. | By name, a yes/no flag (`N` for no). |
| `total_incentive_fee_amount` | Monthly storage fee incentive that applies to this ASIN. | Positive. Already taken off `estimated_monthly_storage_fee`; don't take it off again. |
| `breakdown_incentive_fee_amount` | Description of the incentive type. | `--`, or the incentive's code, a colon and an amount, all inside the text. |
| `average_quantity_customer_orders` | Daily average units ordered by customers but not yet shipped. | Decimal. |

Documented but not in the file: `storage_rate` and `dangerous_goods_storage_type`.

## Personal data

None: no buyer or business details. Rule: strip personal data before analysis and never copy it into anything shared.

## Questions it answers

- Which ASINs cost the most to store last month, and in which country's fulfilment centres? (Add up every row per ASIN, one currency at a time, and go by `fulfillment_center` for the centre's country.)
- Is the storage utilisation surcharge being charged, and on which size tier (`utilization_surcharge_rate`, `storage_utilization_ratio`)?
- Which slow sellers cost more in storage per month than they earn, so they should be removed or repriced?
- How much did storage cost move between months, given that rates change with the time of year?
- Which ASINs got a storage incentive (`total_incentive_fee_amount`)?

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **The month in progress comes back cancelled.** Amazon charges storage after the month ends, so a month still running has no rows yet. Ask again once the month is over.
- **Heavily throttled:** about 6 `createReport` requests a day for this report type before HTTP 429 (see Getting it through the API).
- **`fulfillment_center` comes in single quotes.** Strip them.
- **Several rows per ASIN per centre,** and per FNSKU per centre, some of them exact copies of each other. Keying on ASIN and centre undercounts the fee. The sum of all rows, copies included, matches the bill (next point).
- **The fee is after incentives.** For each month and country checked, the sum of `estimated_monthly_storage_fee` equalled the storage `Base` billed in the next month's `transactions` exactly, and the storage `Base fee` rows in `settlements` matched too. Where an incentive applied, adding it back broke the match. Per row without a utilisation surcharge, fee + incentive equals the undiscounted fee, (`average_quantity_on_hand` - `average_quantity_pending_removal`) x `item_volume` x `base_rate`, to within rounding. That formula is the base fee only: where a utilisation surcharge applies, Amazon's fee also includes the surcharge ([Monthly storage fees](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200612770)), so the check holds only on rows without one. Where the incentive covers the whole fee, the fee, `est_base_msf` and `estimated_total_item_volume` are all `0` though stock was on hand. `est_base_msf` + `est_sus` are after the incentive too, so they can't show the fee before it.
- **Scientific notation.** Small values in the volume and fee columns (`item_volume`, `estimated_total_item_volume`, `est_vol_for_sus`, `est_base_msf`, `est_sus`, `estimated_monthly_storage_fee`, `total_incentive_fee_amount`) sometimes come in the form `4.0E-4`. Read them as numbers.
- **Volumes mix units:** cubic feet for UK rows, cubic metres for EU rows, in the same file.
- **Stock in euro countries is billed in euros.** Rows for euro-country stores are in `EUR`; don't add them to the `GBP` rows.
- Lines end in CRLF, but the last line of the file has no line ending at all, so `wc -l` counts one row short and a shell `while read` loop skips that row. The text is UTF-8.

Sources: [SP-API FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba) (column names) and [Monthly Storage Fees report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G202086720) (meanings).
