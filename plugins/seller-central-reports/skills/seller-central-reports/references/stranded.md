# Stranded Inventory (`stranded`)

Amazon's "FBA Stranded Inventory Report": FBA stock with no active offer (stranded), per SKU, with the reason and the recommended action, as it stands now and updated in near real-time ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). Stranded stock sits in a fulfilment centre without an active offer, for example because the listing was suppressed or never created, and runs up storage fees but can't sell ([Stranded inventory report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201835130)).

## Where to find it in Seller Central

**Supply chain > Supply chain reports** (the Amazon Fulfilment Reports page; [Amazon Fulfilment reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201104750)), then **Inventory > Stranded Inventory** in the left-hand list (`/reportcentral/STRANDED_INVENTORY_UI/1`; **Show more...** expands the Inventory list). **Bulk Fix Stranded Inventory**, lower in the same list, is a different report. Classic: **Reports > Fulfilment by Amazon**. Checked in Seller Central UK on 28 Oct 2026. Amazon's [Inventory reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200885390) help page lists it too.

The page for fixing stranded stock (not the report) is **Supply chain > Manage Amazon inventory**, **Manage stranded inventory** tab (`/inventoryplanning/stranded-inventory`) ([Fix stranded inventory](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201436600)).

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_STRANDED_INVENTORY_UI_DATA` and the marketplace ids of every store the account sells in (from Sellers `getMarketplaceParticipations`, where `isParticipating` is true and `domainName` starts `www.amazon.`, which leaves out the Non-Amazon entries), with no `dataStartTime` or `dataEndTime` and no report options; then poll `getReport` until it's done and fetch the file with `getReportDocument`, unzipping it if Amazon says GZIP. Amazon's [Request a report](https://developer-docs.amazon/sp-api/docs/request-a-report) page says the file covers every store sent unless the report type says otherwise; the [FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba) page says nothing otherwise for this one.
- **Role:** Amazon Fulfillment or Pricing, either one is enough ([Role mappings for types](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)).
- **How far back Amazon goes:** not stated in Amazon's docs. The report shows stock stranded now, with no date range to ask for ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)); `date-stranded` shows how long each item has been stranded. Amazon keeps no past snapshots, so history starts with the first copy you save.

## Columns

Checked against a real UK seller's file in Oct 2026: 18 columns, in this order. Tab-separated text, header row first.

| Column | Meaning | Format and traps |
|---|---|---|
| `primary-action` | The action that takes the stock out of stranded status. | e.g. `Create removal order`. Can be blank. |
| `date-stranded` | When the stock became stranded, without an active offer. | MM/DD/YYYY. |
| `Date-to-take-auto-removal` | Not defined in Amazon's docs; by name, when Amazon will remove the stock automatically. | MM/DD/YYYY. Capitalised, unlike the other names. |
| `status-primary` | Whether the offer is active, inactive or incomplete (missing price or condition). | e.g. `Incomplete`. |
| `status-secondary` | For inactive offers, the reason, e.g. blocked or out of stock. | Can be blank. |
| `error-message` | Extra detail for some inactive offers, e.g. quality alerts. | Can be blank. |
| `stranded-reason` | Not defined in Amazon's docs; why the stock is stranded. | Plain text, not a code, e.g. `Item merged`. |
| `asin` | Amazon's 10-character product identifier. | |
| `sku` | Your SKU. | `£` comes as `&pound;`. |
| `fnsku` | Amazon's identifier for the unit stored in and fulfilled from its fulfilment centres. | |
| `product-name` | The product name. | Can be blank (see Traps). |
| `condition` | The product's condition. | Can be blank. |
| `fulfilled-by` | Whether the product is listed for fulfilment by Amazon or by you. | e.g. `Amazon`. Can be blank. |
| `fulfillable-qty` | Units in fulfilment centres that can be picked, packed and shipped. | Whole number. |
| `your-price` | Your current selling price. | Can be blank. |
| `unfulfillable-qty` | Units in fulfilment centres in unsellable condition. | |
| `reserved-quantity` | Units being picked, packed and shipped, or sidelined for measurement, sampling or other internal processes. | |
| `inbound-shipped-qty` | Units shipped to Amazon and in transit. | |

Every column Amazon lists is in the file; the US help page's `Program` isn't.

## Personal data

None: no buyer or business details. Some sellers put their cost price in the SKU, so treat SKUs as private. Rule: strip personal data before analysis and never copy it into anything shared.

## Questions it answers

- Which SKUs are stranded, since when, and why?
- How many units are stuck, and what are they worth at your price? (`your-price` can be blank.)
- What does Amazon say to do with each one: relist, remove, or switch to FBA?
- Which stranded units does Amazon plan to remove itself, and when (`Date-to-take-auto-removal`)?
- Is stranded stock growing or shrinking week on week, comparing saved snapshots?

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Blank product name and condition.** Take the title from `inventory_planning`, which has stranded SKUs with their titles. Stranded SKUs, FNSKUs and ASINs aren't in `inventory_all` (stranded stock is left out of it). A stranded SKU's row in the UK `listings_all` file can have a blank `item-name` too.
- **Already counted elsewhere.** A stranded SKU's units also show in its `GB` row in `inventory_by_country` (equal to its `fulfillable-qty`), and as `available` in `inventory_planning`. Don't add this report's units to either.
- Dates are MM/DD/YYYY. No country column.
- `£` in SKUs comes as `&pound;`.
- Lines end in CRLF, but the last line of the file has no line ending at all, so `wc -l` counts one row short and a shell `while read` loop skips that row. The text is UTF-8.

Sources: [FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba) (column list), [Stranded inventory report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201835130) (meanings), [Fix stranded inventory](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201436600) (system SKUs).
