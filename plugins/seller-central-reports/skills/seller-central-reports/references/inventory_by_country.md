# FBA Multi-Country Inventory (`inventory_by_country`)

Amazon's "FBA Multi-Country Inventory Report": FBA units available for local fulfilment, per SKU per European country, as they stand now and updated in near real-time, for Multi-Country Inventory sellers in Europe, EU stores only ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). It shows only in-stock, sellable products, leaving out out-of-stock and unsellable units ([Multi-Country Inventory Report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201491370)), but its count is more than the units ready to pick: units moving between centres, inbound units customers can already buy and stranded units (which can't sell) are in it too.

## Where to find it in Seller Central

**Supply chain > Supply chain reports** (the Amazon Fulfilment Reports page; [Amazon Fulfilment reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201104750)), then **Inventory > Multi-Country Inventory** in the left-hand list (`/reportcentral/AFNInventoryReportByCountry/1`; **Show more...** expands the Inventory list). Classic: **Reports > Fulfilment by Amazon**. Checked in Seller Central UK on 28 Oct 2026. The [help page](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201491370) says it is download-only, with no online view.

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_AFN_INVENTORY_DATA_BY_COUNTRY` and the marketplace ids of every store the account sells in (from Sellers `getMarketplaceParticipations`, where `isParticipating` is true and `domainName` starts `www.amazon.`, which leaves out the Non-Amazon entries), with no `dataStartTime` or `dataEndTime` and no report options; then poll `getReport` until it's done and fetch the file with `getReportDocument`, unzipping it if Amazon says GZIP. Amazon's [Request a report](https://developer-docs.amazon/sp-api/docs/request-a-report) page says the file covers every store sent unless the report type says otherwise; the [FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba) page says nothing otherwise for this one.
- **Role:** Amazon Fulfillment or Pricing for the report, either one is enough ([Role mappings for types](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)). The store list from `getMarketplaceParticipations` needs Product Listing or Selling Partner Insights in the EU ([role mappings](https://developer-docs.amazon/sp-api/docs/role-mappings)); without either, send the marketplace ids yourself.
- **How far back Amazon goes:** not stated in Amazon's docs. The report is a snapshot of stock when it runs, with no date range to ask for ([Multi-Country Inventory Report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201491370)). Amazon keeps no past snapshots, so history starts with the first copy you save.

## Columns

Checked against a real UK seller's file in Oct 2026: 6 columns, in this order. Tab-separated text, header row first. One row per SKU per country with stock.

| Column | Meaning | Format and traps |
|---|---|---|
| `seller-sku` | Your SKU. | HTML entities: `£` comes as `&pound;`, `&` as `&amp;`, `–` as `&ndash;`. Decode them before joining to `inventory_all`, which has plain characters. |
| `fulfillment-channel-sku` | Amazon's identifier for the unit stored in and fulfilled from its fulfilment centres (the FNSKU). | |
| `asin` | Amazon's 10-character product identifier. | |
| `condition-type` | The condition of the stock. | Condition codes such as `NewItem`. |
| `country` | Country code of where the stock is held. | Two letters. Can hold codes beyond the five countries Amazon's help lists (DE, ES, FR, GB and IT), so don't hard-code that list. |
| `quantity-for-local-fulfillment` | Units available to fulfil orders in the country where the customer is. | Whole number. **Not just fulfillable units.** For `GB` SKUs also in `inventory_all` (joined after decoding every entity), it usually equals `afn-fulfillable-quantity` + `afn-fc-transfer-quantity` + `afn-future-supply-buyable`, and far less often `afn-fulfillable-quantity` alone. Where a SKU has `afn-future-supply-buyable` but nothing inbound, it leaves that out: it counts it only where units are inbound. A stranded SKU's `GB` row equals its `fulfillable-qty` in `stranded`. |

Every column Amazon lists is in the file.

## Personal data

None: no buyer or business details. Some sellers put their cost price in the SKU, so treat SKUs as private. Rule: strip personal data before analysis and never copy it into anything shared.

## Questions it answers

- How many units of each SKU are held in each EU country? Use the non-`GB` rows. Never add `storage_fees` quantities to them: those are daily averages for a past month (the month in progress comes back cancelled), so adding them can count stock that has gone. Use `storage_fees` only to flag SKUs to look into, such as EU stock billed there but missing here (see Traps). For the UK, use `inventory_all`'s `afn-fulfillable-quantity` for units ready to pick: the `GB` rows here also count units in transfer, inbound units customers can already buy and stranded units. Don't add `stranded` to this report: its units are already here.
- Which SKUs have no local stock in a given country?
- Is stock spread across countries the way you expect?
- How did per-country stock change week on week, comparing saved snapshots?

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Codes outside Amazon's list.** `country` can hold codes beyond the five Amazon's help lists (DE, ES, FR, GB and IT). Don't hard-code that list.
- **EU stock can be missing.** EU-held stock can be billed in `storage_fees` but be missing from this report and from every other stock report (`inventory_all`, `restock`, `inventory_planning`). It isn't always the unsellable stock this report leaves out, and Amazon doesn't say why it happens. Otherwise the FNSKUs here outside `GB` match the ones `storage_fees` billed outside `GB` in the latest month, with the same units. Cross-check EU stock against `storage_fees`, and look up any FNSKU billed there but missing here in `ledger_detail`; don't add its billed units to the stock count (see Questions it answers).
- **Stranded units are counted.** A SKU in `stranded` has a `GB` row here equal to its `fulfillable-qty`, though stranded stock can't sell. Stranded SKUs aren't in `inventory_all`.
- **HTML entities in SKUs:** `&pound;`, `&amp;` and `&ndash;`. Decode them before joining to other reports.
- Lines end in CRLF, but the last line of the file has no line ending at all, so `wc -l` counts one row short and a shell `while read` loop skips that row. The text is UTF-8.

Sources: [FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba) (column list), [Multi-Country Inventory Report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201491370) (meanings).
