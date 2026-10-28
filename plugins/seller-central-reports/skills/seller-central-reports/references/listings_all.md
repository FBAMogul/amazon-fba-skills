# All listings report (`listings_all`)

Amazon's "detailed all listings report": every listing in one store, with its SKU, ASIN, price, quantity, fulfilment channel and status, as it stands on the day you ask for it ([SP-API inventory reports](https://developer-docs.amazon/sp-api/docs/report-type-values-inventory)).

## Where to find it in Seller Central

Checked in Seller Central UK on 28 Oct 2026: **Products > Product reports**, which opens the Inventory reports page. In classic Seller Central it is **Reports > Inventory Reports**. Pick **All Listings Report** in the **Request an Inventory report** drop-down. Amazon describes it as your items "including all listings regardless of status" and says it can take up to 45 minutes. A **Customise the columns for this report** link changes its columns. The list of requested reports below also shows reports that apps connected to the account asked for through the API.

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_MERCHANT_LISTINGS_ALL_DATA` and no `dataStartTime`/`dataEndTime`; then `getReport` and `getReportDocument`. Amazon accepts only one store per report: given several marketplace ids, it uses only the first ([SP-API inventory reports](https://developer-docs.amazon/sp-api/docs/report-type-values-inventory)). So ask store by store, each with that store's marketplace id only (the UK is `A1F83G8C2ARO7P`, Germany `A1PA6795UKMFR9`: [Store Identifiers](https://developer-docs.amazon/sp-api/docs/store-identifiers)).
- **Report options:** Amazon accepts `custom` and `preferredReportDocumentLocale` (the language of the column headers) ([SP-API inventory reports](https://developer-docs.amazon/sp-api/docs/report-type-values-inventory)).
- **Role:** any one of Inventory and Order Tracking, Pricing, Product Listing, or Direct to Consumer Shipping (restricted) ([role mappings](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)).
- **How far back Amazon goes:** not stated in Amazon's docs. Each file is the listings as they stood when asked for. Amazon can't make one for a past date, so history starts with the first copy you save. A copy your app requested in the last 90 days can still be fetched (`getReports`, then `getReportDocument`).

## Columns

Checked against a real UK seller's files (Oct 2026), one per store. The file is tab-separated text with a header row first. It starts with a UTF-8 byte order mark, and lines end in LF, not CRLF. Headers are in English in every store, but values such as `merchant-shipping-group` are in the store's language.

**The header can differ by store.** The full file has the 31 columns below. A store's file can have only 18 (no `item-description`, `image-url`, `item-is-marketplace`, `zshop-…`, `asin1` to `asin3`, `bid-for-featured-placement` or `add-delete`), and a file for a store with no listings has 28 (no `merchant-shipping-group`, `Minimum order quantity` or `Sell remainder`). Find columns by name, never by position.

Amazon defines none of these columns; meanings marked "by name" are read from the name and the values.

| Column | Meaning | Format and traps |
|---|---|---|
| `item-name` | By name, the listing title. | Can be blank. |
| `item-description` | By name, the description. | Blank in every file checked. |
| `listing-id` | By name, Amazon's id for the listing. | 11 characters. |
| `seller-sku` | Your SKU. | |
| `price` | By name, your price. | In the store's currency (GBP in the UK). Can be blank. |
| `quantity` | By name, the seller-fulfilled quantity. | Blank on FBA listings; filled only on seller-fulfilled (`DEFAULT`) rows. |
| `open-date` | By name, when the listing was created. | **DD/MM/YYYY HH:MM:SS** followed by a local time zone name (`BST` or `GMT` in the UK, `MEST` in EU stores). Not the MM/DD of the FBA reports. |
| `image-url` | By name, the main image. | Blank in every file checked. |
| `item-is-marketplace` | Not defined. | Can be `y` on every row. |
| `product-id-type` | By name, the type of `product-id`. | Can be `1` (ASIN) on every row. |
| `zshop-shipping-fee` | Legacy zShops field. | Blank in every file checked. |
| `item-note` | By name, the seller's condition note. | Can be blank. |
| `item-condition` | By name, the condition as a number. | Condition codes such as `11`, or blank. The numbers line up with [`inventory_all`](inventory_all.md)'s condition names: for example `1` is `UsedLikeNew` and `500` is `Unknown`. `500` doesn't reliably mark Grade and Resell: Grade and Resell (`amzn.gr.`) SKUs can carry ordinary used codes, and `500` rows needn't be Grade and Resell. |
| `zshop-category1`, `zshop-browse-path`, `zshop-storefront-feature`, `zshop-boldface` | Legacy zShops fields. | Blank in every file checked. |
| `asin1` | By name, the ASIN. | Same as `product-id`. |
| `asin2`, `asin3` | Not defined. | Blank in every file checked. |
| `will-ship-internationally` | Not defined. | Blank in every file checked. |
| `expedited-shipping` | Not defined. | Blank in every file checked. |
| `product-id` | By name, the product id of type `product-id-type`. | The ASIN. |
| `bid-for-featured-placement` | Legacy field. | Blank in every file checked. |
| `add-delete` | Legacy feed field. | Blank in every file checked. |
| `pending-quantity` | Not defined. | Filled only on seller-fulfilled (`DEFAULT`) rows. |
| `fulfillment-channel` | By name, who fulfils the listing. | `AMAZON_EU` (FBA) or `DEFAULT` (seller-fulfilled). |
| `merchant-shipping-group` | By name, the shipping template. | `Migrated Template` in the UK; in the store's language elsewhere. |
| `status` | By name, the listing status. | `Active`, `Inactive` or `Incomplete`. |
| `Minimum order quantity` | Not in Amazon's docs. | Blank in every file checked. Capitalised, with spaces. |
| `Sell remainder` | Not in Amazon's docs. | Blank in every file checked. |

Every column on Amazon's list is in the file; `Minimum order quantity` and `Sell remainder` are extra.

Source: [SP-API inventory reports](https://developer-docs.amazon/sp-api/docs/report-type-values-inventory) (column names only).

## Downloaded from Seller Central

**The same file as the API's.** Seen in a real UK seller's Seller Central download (Oct 2026), one store: tab-separated, UTF-8 with a byte order mark, lines ending in LF, and the same 31 columns as above, from `item-name` to `Sell remainder`. Special characters in SKUs come through as plain text (a `£` is `£`). Seller Central offers no CSV for this report.

The file is named `All+Listings+Report_MM-DD-YYYY.txt`, by the day you download it (in US date order), not by anything in it. Customising the columns may change the file, so read columns by name. For traps shared by all Seller Central downloads, see [seller-central-downloads](seller-central-downloads.md).

## Personal data

None. No buyer or business personal details.

## Questions it answers

- What status does each SKU have in each store (`status`), and which ones are not live?
- Which SKUs are FBA and which FBM in each store (`fulfillment-channel`)?
- Which SKUs are listed in the UK but missing from an EU store, or the other way round? Compare each store's file from the same day.
- Which listings are old (`open-date`) or not active, and still hold FBA stock? This file has no FBA stock column, so start from the stock reports and look each SKU up here, in the store that holds the stock; [`inventory_all`](inventory_all.md)'s page gives the method. Don't use `inventory_all`'s `afn-total-quantity`: it counts inbound units, so a SKU can show stock there that is only inbound. And `inventory_all` misses SKUs that hold stock with no active listing (stock in EU stores, stranded stock, SKUs with only reserved units or only unsellable units). Those SKUs often have no row in this file: count a SKU with no row here as not listed, and so not active, since an inner join drops them.

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Headers can differ by store** (31, 28 or 18 columns). Read by column name, never by position.
- **`open-date` is DD/MM/YYYY** with a local time zone name: `BST` or `GMT` in the UK, `MEST` in EU stores. The FBA reports use MM/DD.
- **Inactive listings are in too.** The file holds old listings, not just live ones, so filter on `status`.
- **Grade and Resell offers show up** as `amzn.gr.` SKUs, which can have condition `500` and status `Incomplete`.
- **`item-condition` `500` is not a Grade and Resell flag**; use the `amzn.gr.` SKU prefix instead.
- **A store with no listings** comes back as a header row only.
- Each file starts with a UTF-8 byte order mark; lines end in LF, not CRLF.
