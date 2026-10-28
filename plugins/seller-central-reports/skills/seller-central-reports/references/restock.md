# Restock Inventory (`restock`)

Amazon's "Restock Inventory Report": restock recommendations per SKU and country (stock levels, days of supply, low and out-of-stock alerts, and how many units to send by when), as they stand now and updated in near real-time, in every store except India ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). Recommendations use your settings such as lead time and case pack quantities ([Restock Inventory report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G202105670)), and count customer demand across all the stores you sell in ([Restock Inventory](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201634550)). For the stock itself, use `inventory_all` (UK; this report can lag it) and `inventory_by_country` (per country).

## Where to find it in Seller Central

**Supply chain > Manage Amazon inventory** opens the **FBA Inventory** page (`/inventoryplanning/manageinventoryhealth`); its **Reports** drop-down has **Restock report** ([FBA Inventory overview](https://sellercentral.amazon.co.uk/help/hub/reference/external/GTMXYZN64UJL7TT6)). The report is also in **Supply chain > Supply chain reports** (the Amazon Fulfilment Reports page), under **Inventory > Restock Inventory** (`/reportcentral/RestockReport/1`; **Show more...** expands the Inventory list). Classic: **Inventory > FBA Inventory** for the page, or **Reports > Fulfilment by Amazon** for the list. Checked in Seller Central UK on 28 Oct 2026. The [Restock Inventory report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G202105670) help page links a download page at `https://sellercentral.amazon.co.uk/restockinventory/reports?reportTypeId=94300`, which wasn't checked.

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_RESTOCK_INVENTORY_RECOMMENDATIONS_REPORT` and the marketplace ids of every store the account sells in (from Sellers `getMarketplaceParticipations`, where `isParticipating` is true and `domainName` starts `www.amazon.`, which leaves out the Non-Amazon entries), with no `dataStartTime` or `dataEndTime` and no report options; then poll `getReport` until it's done and fetch the file with `getReportDocument`, unzipping it if Amazon says GZIP. Amazon's [Request a report](https://developer-docs.amazon/sp-api/docs/request-a-report) page says the file covers every store sent unless the report type says otherwise; the [FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba) page says nothing otherwise for this one.
- **Role:** Amazon Fulfillment or Pricing for the report, either one is enough ([Role mappings for types](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)). The store list from `getMarketplaceParticipations` needs Product Listing or Selling Partner Insights in the EU ([role mappings](https://developer-docs.amazon/sp-api/docs/role-mappings)); without either, send the marketplace ids yourself.
- **How far back Amazon goes:** not stated in Amazon's docs. The report gives current stock and recommendations, with no date range to ask for ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). Amazon keeps no past snapshots, so history starts with the first copy you save.

## Columns

Checked against a real UK seller's file in Oct 2026: 29 columns, in this order. Tab-separated text, header row first.

| Column | Meaning | Format and traps |
|---|---|---|
| `Country` | The country for this SKU. | One row per SKU per country, as a two-letter code (e.g. `GB`). For a UK seller, non-`GB` rows don't show stock held in the EU (see Traps). |
| `Product Name` | The product title. | |
| `FNSKU` | Amazon's fulfilment network SKU. | |
| `Merchant SKU` | Your SKU. | Plain `£`, not `&pound;`. |
| `ASIN` | Amazon's 10-character product identifier. | |
| `Condition` | The unit's condition. | A condition code written as one word, such as `New`; can be `Unknown`. |
| `Supplier` | Supplier name (the supplier nickname). | `--` when no supplier is entered. |
| `Supplier part no.` | The supplier's part number, entered on the SKU detail page. | Blank when none is entered. |
| `Currency code` | The currency used for this SKU. | The currency of the row's country (e.g. `GBP` on `GB` rows), blank where there's no price. |
| `Price` | Your list price. | In the row's `Currency code`. Blank when `Currency code` is. |
| `Sales last 30 days` | Amount sold in the last 30 days. | A money amount, decimal (e.g. `0.0`); units are in the next column. |
| `Units Sold Last 30 Days` | Units sold in the last 30 days. | Whole number. |
| `Total Units` | Amazon says all units in fulfilment centres: available, FC transfer, researching, unfulfillable and reserved. The file counts differently. | **Includes `Inbound`, but not researching or unfulfillable units.** It equals `Available` + `FC transfer` + `FC Processing` + `Customer Order` + `Inbound` on every row. Unfulfillable units are left out, so a SKU holding only unfulfillable units shows `Total Units` `0`, and researching units are left out too. Never add `Inbound` to it, and don't read it as everything Amazon holds. |
| `Inbound` | Units in a shipment Amazon has been told about, has a tracking number for, or has received for processing. | Can lag `inventory_all` (see Traps). |
| `Available` | Units in fulfilment centres that can be picked, packed and shipped. | Can lag `inventory_all` (see Traps). |
| `FC transfer` | Units moving between fulfilment centres. | |
| `FC Processing` | Units sidelined at a fulfilment centre for extra processing. | |
| `Customer Order` | Units reserved for customer orders. | |
| `Unfulfillable` | Units in fulfilment centres that can't be sold. | Not included in `Total Units`. |
| `Working` | Units in shipments with working status. | |
| `Shipped` | Units in shipments with shipped status. | |
| `Receiving` | Units in shipments with receiving status. | |
| `Fulfilled by` | Whether the product is listed for fulfilment by Amazon or by you. | e.g. `Amazon`. |
| `Total Days of Supply (including units from open shipments)` | Not confirmed. Amazon notes it includes units from open shipments. | Amazon's list calls it `Total Days of Supply`. Can be blank. |
| `Days of Supply at Amazon Fulfillment Network` | Estimated days the current stock will last at projected demand. | Can be blank. |
| `Alert` | Whether the product is low or out of stock. | `out_of_stock`, `low_stock` or blank. |
| `Recommended replenishment qty` | Units Amazon recommends sending to FBA. | Can be blank. |
| `Recommended ship date` | Date to ship by to avoid running low or out. | MM/DD/YYYY. Can be blank; where filled, it is often the day the report was fetched. |
| `Unit storage size` | Cube space the units take up. | A number and its unit in one text field: `ft3` on `GB` rows, `m3` on the others. |

Documented but not in the file: `Recommended action` (US only).

## Personal data

None: no buyer or business details. `Supplier` and `Supplier part no.` hold your own supplier details if you've entered them; keep those private too. Some sellers put their cost price in the SKU, so treat SKUs as private. Rule: strip personal data before analysis and never copy it into anything shared.

## Questions it answers

- Which SKUs does Amazon flag as low or out of stock in the UK? Not in EU countries: for a UK seller, this report's EU rows don't show EU stock (see Traps).
- How many units should go in for each SKU, and by what date?
- How many days of supply does each SKU have at Amazon, and with open shipments counted?
- How do 30-day sales compare with stock per SKU? Take the stock from `inventory_all` (`afn-fulfillable-quantity` for pickable UK units, and its inbound columns), not from `Available` and `Inbound` here, and don't mix the two reports: they can disagree. Don't add `Inbound` to `Total Units`: it's already in it.

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Old SKUs listed too.** The file can include SKUs with nothing available, flagged `out_of_stock`, alongside current ones.
- **Not for EU stock.** For a UK seller, this report doesn't reflect stock held in EU fulfilment centres: a non-`GB` row can read `out_of_stock` with every stock column `0`, and a SKU held in the EU can have no row here at all. This report can't answer EU stock: use `inventory_by_country`.
- **Lags `inventory_all`.** Fetched a few minutes after `inventory_all`, it can still show as `Inbound` units that `inventory_all` already has as fulfillable; `ledger_detail` can show their receipt only the next day. `inventory_planning` can lag the same way. Take pickable stock from `inventory_all`.
- **`Total Units` leaves out unfulfillable and researching units** but includes `Inbound` (see Columns).
- **Supplier columns are empty unless you've entered a supplier:** `Supplier` is `--` and `Supplier part no.` blank.
- `Unit storage size` is text with the unit attached: `ft3` for `GB` rows, `m3` for the rest.
- **Prices are in each row's own currency** (`Currency code`), so a file with rows for several countries mixes currencies: total per currency.
- Lines end in CRLF, but the last line of the file has no line ending at all, so `wc -l` counts one row short. The text is UTF-8.

Sources: [FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba) (column list), [Restock Inventory report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G202105670) (meanings), [Restock Inventory](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201634550) (supplier nickname).
