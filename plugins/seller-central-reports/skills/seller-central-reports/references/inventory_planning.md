# Manage Inventory Health (`inventory_planning`)

Amazon's "FBA Manage Inventory Health Report": an inventory health view per SKU, with sales over 7, 30, 60 and 90 days, shipment statuses, stock age, aged and excess units, estimated storage fees and aged inventory surcharges, and recommended actions, as it stands now ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). Columns vary by store, and Amazon says it is not available in the NL, PL, SE or BE stores ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). For pickable stock use `inventory_all`: this report can lag it.

## Where to find it in Seller Central

Checked in Seller Central UK on 28 Oct 2026. Seller Central calls it the **FBA Inventory** report. Two ways in:

- **Supply chain > Manage Amazon inventory** opens the FBA Inventory page. Its **Reports** drop-down has **Inventory report** and Restock report ([FBA Inventory overview](https://sellercentral.amazon.co.uk/help/hub/reference/external/GTMXYZN64UJL7TT6)). Inventory report opens **Supply chain reports > FBA Inventory**.
- **Supply chain > Supply chain reports** opens Amazon Fulfilment Reports. **FBA Inventory** is under Inventory in the left-hand list (**Show more...** expands it) ([Amazon Fulfilment reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201104750)).

In classic Seller Central it is **Inventory > FBA Inventory**, or **Reports > Fulfilment by Amazon**. Amazon's [Inventory reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200885390) page links the report at `https://sellercentral.amazon.co.uk/reportcentral/MANAGE_INVENTORY_HEALTH/1`, with [FBA Inventory report](https://sellercentral.amazon.co.uk/help/hub/reference/external/GKT9YKCHXMSDVJKB) as its help page. Amazon's note on the report page says it brings together sales over 7, 30, 60 and 90 days, shipment statuses, excess units and aged stock with its fees. It replaces the Inventory Health, Inventory Age and Excess Inventory reports, which had "up to a 10-day data delay".

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_FBA_INVENTORY_PLANNING_DATA` and the marketplace ids of every store the account sells in (from Sellers `getMarketplaceParticipations`, where `isParticipating` is true and `domainName` starts `www.amazon.`, which leaves out the Non-Amazon entries), with no `dataStartTime` or `dataEndTime` and no report options; then poll `getReport` until it's done and fetch the file with `getReportDocument`, unzipping it if Amazon says GZIP. Amazon's [Request a report](https://developer-docs.amazon/sp-api/docs/request-a-report) page says the file covers every store sent unless the report type says otherwise. Amazon doesn't say what happens when NL, PL, SE or BE are among the stores sent.
- **Role:** Amazon Fulfillment only for the report; Pricing isn't enough for this one ([Role mappings for types](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)). The store list from `getMarketplaceParticipations` needs Product Listing or Selling Partner Insights in the EU ([role mappings](https://developer-docs.amazon/sp-api/docs/role-mappings)); without either, send the marketplace ids yourself.
- **How far back Amazon goes:** not stated in Amazon's docs. The report is a current view, with no date range to ask for; its sales columns look back up to 90 days and its age columns cover stock held for over a year ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). Amazon can't make one for a past date, so history starts with the first copy you save. A copy your app requested in the last 90 days can still be fetched (`getReports`, then `getReportDocument`).

## Columns

Checked against a real UK seller's file in Oct 2026: 89 columns, in this order. Tab-separated text, header row first. The names match the API list (`inv-age-…`, `sales-price`), not the help page's spellings.

**`-1` means no data.** Rows with `pending-removal-quantity` `-1` and `sell-through` `-1.0` have blank age columns. They can be SKUs with no stock, but some still hold stock (inbound or available units). Filter them out before averaging, but not before counting stock.

| Column | Meaning | Format and traps |
|---|---|---|
| `snapshot-date` | The date the report was generated. | A date, not a date-time: YYYY-MM-DD. |
| `sku` | Your SKU. | One row per SKU. |
| `fnsku` | Amazon's identifier for the unit stored in and fulfilled from its fulfilment centres. | |
| `asin` | Amazon's 10-character product identifier. | |
| `product-name` | The product name. | Can be blank. |
| `condition` | The unit's condition. | Often blank; a condition such as `New` where filled. |
| `available` | Available (sellable) units. | Counts stranded units. Can lag `inventory_all` (see Traps). |
| `fc-transfer` | Not defined in Amazon's docs; units moving between fulfilment centres. | |
| `pending-removal-quantity` | Units you have asked to be returned or disposed of. | `-1` = no data. |
| `inv-age-0-to-90-days`, `inv-age-91-to-180-days`, `inv-age-181-to-270-days`, `inv-age-271-to-365-days` | Sellable units held in fulfilment centres for that many days. | Blank on rows with no age data. Overlap the finer bands further down: `0-to-90` = `0-to-30` + `31-to-60` + `61-to-90`, and `181-to-270` + `271-to-365` = `181-to-330` + `331-to-365`. |
| `inv-age-366-to-455-days`, `inv-age-456-plus-days` | Not defined on the help page; by name, sellable units held for that many days. | Blank on rows with no age data. Come straight after the four bands above. |
| `currency` | The currency for the estimated aged inventory surcharge. | e.g. `GBP`. |
| `units-shipped-t7`, `units-shipped-t30`, `units-shipped-t60`, `units-shipped-t90` | Units shipped in the last 7, 30, 60, 90 days. | Often blank. |
| `alert` | Shown when an item has a low-traffic or low-conversion alert. | `Low conversion`, `Low traffic` or blank. |
| `your-price` | Your list price. | `0.0` when there's no offer. |
| `sales-price` | Your sale price, if a sale is running. | Decimal; can be `0.0`. |
| `lowest-price-new-plus-shipping` | Lowest price, delivery included, for the item new on Amazon. | Decimal; can be `0.0`. |
| `lowest-price-used` | Lowest price, delivery included, for the item used on Amazon. | Decimal; can be `0.0`. |
| `recommended-action` | Amazon's recommended action. | Values include `Edit listing`, `GoToRestock`, `NoExcessInventory`, `Lower price`, `Create sale`, `Create Outlet deal`, `Advertise listing`, `GoToStranded`. |
| `healthy-inventory-level` | Deprecated; replaced by `fba-minimum-inventory-level` and `fba-inventory-level-health-status`. | Named without the `DEPRECATED` prefix Amazon's list shows. Often blank. |
| `recommended-sales-price` | Price Amazon recommends to sell through excess stock. | Often blank. |
| `recommended-sale-duration-days` | If a sale is recommended, how many days it should run. | Whole number; `0` when none. |
| `recommended-removal-quantity` | Units likely better removed than charged storage fees. | Can be `0` on every row, so don't read a zero as advice to keep stock. |
| `estimated-cost-savings-of-recommended-actions` | Estimated saving from the recommended action against storage fees from doing nothing. | Often blank. |
| `sell-through` | Units sold and shipped over the past 90 days divided by average available units over that time. | Decimal. `-1.0` = no data. |
| `item-volume` | Item volume: longest × median × shortest side. | Often blank. |
| `volume-unit-measurement` | Unit of `item-volume`. | |
| `storage-type` | Storage type for storage limits. | A storage type name, e.g. `Standard`. |
| `storage-volume` | Total storage the product uses, minus stock pending removal. | |
| `marketplace` | The Amazon store where the product is located. | `UK` for the UK store, not `GB`. |
| `product-group` | Amazon's product grouping. | Amazon's internal codes starting `gl_`, not the names in `fba_fee_preview`. |
| `sales-rank` | The product's current sales rank. | Can be blank. |
| `days-of-supply` | Not defined in Amazon's docs. | Often blank. |
| `estimated-excess-quantity` | Overstocked units that cost more to keep than to clear. | Whole number. |
| `weeks-of-cover-t30`, `weeks-of-cover-t90` | Trailing 30- or 90-day average on-hand stock divided by units sold in that time, in weeks. | Rarely filled. |
| `featuredoffer-price` | Price of the Featured Offer (the offer with the Add to Basket button). | Filled even for SKUs with no stock. |
| `sales-shipped-last-7-days`, `sales-shipped-last-30-days`, `sales-shipped-last-60-days`, `sales-shipped-last-90-days` | Sales value of units shipped in the last 7, 30, 60, 90 days. | Money amounts (GBP for the UK store), not unit counts; units are in `units-shipped-t…`. Often blank. |
| `inv-age-0-to-30-days`, `inv-age-31-to-60-days`, `inv-age-61-to-90-days` | Sellable units held in fulfilment centres for that many days. | Blank on rows with no age data. Together they equal `inv-age-0-to-90-days`: don't add both. |
| `inv-age-181-to-330-days` | Sellable units held for 181 to 330 days. | Filled (often `0`) on every row, unlike the other age bands. With `331-to-365` it equals `181-to-270` + `271-to-365`. |
| `inv-age-331-to-365-days` | Sellable units held for 331 to 365 days. | Blank on rows with no age data. |
| `estimated-storage-cost-next-month` | Estimated storage fees 30 days from now at current sales. | Often blank. |
| `inbound-quantity` | Inbound working + shipped + received. | Can lag `inventory_all` (see Traps). |
| `inbound-working`, `inbound-shipped`, `inbound-received` | Units in shipments with working, shipped or receiving status. | Can be blank. |
| `Total Reserved Quantity` | Units in reserved status. | |
| `unfulfillable-quantity` | Units in stock that can't be sold or fulfilled. | |
| `quantity-to-be-charged-ais-241-270-days`, `estimated-ais-241-270-days`, and the same pair for `271-300`, `301-330`, `331-365`, `366-455` and `456-plus` | Available units that will be in that age band on the next charge date, and the estimated aged inventory surcharge for them. | Often blank. Currency in `currency`. |
| `historical-days-of-supply` | Not defined in Amazon's docs. | Rarely filled. |
| `Recommended ship-in quantity` | Not defined in Amazon's docs. | Can be blank. |
| `Recommended ship-in date` | Not defined in Amazon's docs. | Often blank. |
| `Last updated date for Historical Days of Supply` | Updated date of the low-inventory-level fee. | |
| `Short term historical days of supply` | Historical days of supply over the last 30 days. | |
| `Long term historical days of supply` | Historical days of supply over the last 90 days. | |
| `Inventory age snapshot date` | The date inventory age and aged inventory surcharges are worked out on. | YYYY-MM-DD, filled only on rows with age data. Can be a few days before `snapshot-date`. |
| `Inventory Supply at FBA` | Amazon's help: units in fulfilment centres that can be picked, packed and shipped. In the file it's more. | Equals `available` + `fc-transfer` + `inbound-quantity` on every row. For pickable UK stock use `inventory_all`'s `afn-fulfillable-quantity`, not this or `available` (see Traps). |
| `Reserved FC Processing` | Units sidelined at a fulfilment centre for extra processing. | |
| `Reserved Customer Order` | Units reserved for customer orders. | |
| `Reserved Staging` | Units Amazon holds briefly to keep fulfilment fast (per the Reserved Inventory report). | Can be `0` on every row. |
| `Total Days of Supply (including units from open shipments)` | Not defined in Amazon's docs. | Often blank. |
| `is-seasonal-in-next-3-months` | Not defined in Amazon's docs. | `Y` or `N`. |
| `season-name` | Not defined in Amazon's docs. | The season's name, or blank. |
| `season-start-date`, `season-end-date` | Not defined in Amazon's docs. | Often blank. |
| `Fulfillment Service and Programs` | Programmes the item is in. | Short codes, separated by a comma and a space. Last column. |

Documented but not in the file: `inv-age-365-plus-days`, `no-sale-last-6-months`, `quantity-to-be-charged-ais-181-210-days`, `-211-240-days` and `-365-plus-days` with their `estimated-ais-…` pairs, `fba-minimum-inventory-level`, `fba-inventory-level-health-status`, `Exempted from Low-Inventory-Level fee?`, `Low-Inventory-Level fee applied in current week?` and `supplier`. None of the extra help-page columns appear either.

## Downloaded from Seller Central

**The same columns as the API's, in a different format.** Seen in a real UK seller's Seller Central download (Oct 2026), as .csv: the same 89 columns, with the same names in the same order as the API file. The differences:

- **Comma-separated**, not tab-separated.
- **Starts with a UTF-8 byte order mark.** Strip it, or the first header won't match `snapshot-date`.
- **Every value is in double quotes.** How a `"` inside a value is written wasn't checked.
- Special characters in SKUs come through as plain text (a `£` is `£`).

The page also offers .txt, which wasn't checked. Amazon warns on every Fulfilment report page that opening the .csv in a spreadsheet drops leading zeros from SKUs, ASINs and FNSKUs, and suggests .txt to keep them. For that and the other traps shared by all Seller Central downloads, see [seller-central-downloads](seller-central-downloads.md).

## Personal data

None: no buyer or business details. Some sellers put their cost price in the SKU, so treat SKUs as private. Rule: strip personal data before analysis and never copy it into anything shared.

## Questions it answers

- Which SKUs have stock in the aged inventory surcharge bands, and what's the estimated surcharge (`quantity-to-be-charged-ais-…` and `estimated-ais-…`, from 241 days on; the file has no 181 to 240 day bands)?
- Which SKUs carry excess units, and what does Amazon recommend (`recommended-action`: edit listing, restock, lower price, create a sale or Outlet deal, advertise)? `recommended-removal-quantity` can be `0` on every row.
- What's each SKU's sell-through and weeks of cover?
- How old is each SKU's stock (the `inv-age-…` bands)? The bands overlap, so add up one set only: either `0-to-90`, `91-to-180`, `181-to-270`, `271-to-365`, `366-to-455` and `456-plus`, or `0-to-30`, `31-to-60`, `61-to-90`, `91-to-180`, `181-to-330`, `331-to-365`, `366-to-455` and `456-plus`.
- What storage cost does Amazon estimate for next month, per SKU?

The minimum inventory level and the low-inventory-level fee aren't in this report; check those in Seller Central.

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **UK rows only.** A UK account's file can hold only `marketplace` `UK` rows, even when the request names EU stores too.
- **Rows with no stock.** The file can list SKUs with no stock. Rows with `-1` in `pending-removal-quantity` and `sell-through` have no age data, though some of them still hold stock. Inventory age is filled mainly on rows with available stock.
- **Inventory age is a few days old:** `Inventory age snapshot date` was a few days before `snapshot-date`.
- **Stock can lag `inventory_all`.** Fetched a few minutes after `inventory_all`, it still showed as inbound units that `inventory_all` had as fulfillable; `ledger_detail` had their receipts the next day. `restock` lagged the same way. Take pickable stock from `inventory_all`, and don't add inbound from this report to another's stock.
- **Holds FNSKUs `inventory_all` doesn't.** Some FNSKUs here weren't in `inventory_all`, and some of those held stock: stranded SKUs (their units are in `available`), FNSKUs with only reserved units and FNSKUs with only unsellable units. Use this report for those.
- **`recommended-removal-quantity` and `Reserved Staging` can be `0` on every row,** so don't treat a zero as Amazon's advice.
- **Missing columns:** the low-inventory-level fee flags, `fba-minimum-inventory-level`, `fba-inventory-level-health-status`, `no-sale-last-6-months` and the 181-240 day surcharge bands aren't in the file.
- Lines end in CRLF, but the last line of the file has no line ending at all, so `wc -l` counts one row short and a shell `while read` loop skips that row. The text is UTF-8.

Seen in Seller Central downloads (Oct 2026):

- **The .csv is not the API's layout.** It is comma-separated, quotes every value and starts with a byte order mark, so a script written for the API's tab-separated file won't read it as it is (see Downloaded from Seller Central).

Sources: [FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba) (column list; Brazil-only columns left out), [FBA Inventory report](https://sellercentral.amazon.co.uk/help/hub/reference/external/GKT9YKCHXMSDVJKB) (meanings).
