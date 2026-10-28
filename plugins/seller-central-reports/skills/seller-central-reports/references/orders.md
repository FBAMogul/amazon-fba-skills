# All Orders report (`orders`)

Every order placed in the period you ask for, FBA and seller-fulfilled, one row per order item, chosen by purchase date: Amazon's Flat File Orders By Order Date report, "all orders that were placed in the specified period" whatever the fulfilment channel or shipment status ([Amazon](https://developer-docs.amazon/sp-api/docs/report-type-values-order#order-tracking-reports)), which Seller Central calls the All Orders Report ([UK help](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200547170)).

## Where to find it in Seller Central

Checked in Seller Central UK on 28 Oct 2026: **Orders > Order reports**, then the **All orders** tab. In classic Seller Central it is **Orders > Order Reports**. The other tabs are FBM orders, FBM unshipped orders, Archived Orders (for orders more than two years old) and End of day forms.

Amazon's text on the tab says the report covers FBA and seller-fulfilled orders from the past two years, whatever their status, and each download covers at most 30 days. Choose **Order date** as the type of report: that picks orders by purchase date, like this page's report (**Last Updated Date** picks them by their last change). Then choose a date range (the last day, 3, 7, 14 or 30 days, or exact dates in boxes that take **MM/DD/YYYY**) and request it. Amazon says it can take up to 45 minutes. The list of requests showed each report's date range covered in British Summer Time and the request and completion times in GMT. An **Add or remove order report columns** link changes the columns. Seller Central offers no XML version: Amazon's banner says to request that through the API.

Amazon's UK help page says there is no online view of this report, only the download ([UK help](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200547170)).

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_FLAT_FILE_ALL_ORDERS_DATA_BY_ORDER_DATE_GENERAL`, the marketplace ids of the stores you want, and `dataStartTime`/`dataEndTime` set to the period; then `getReport` until the report is done, and `getReportDocument` (unzip it if Amazon says GZIP). No report options.
- **Role:** any one of Amazon Fulfillment, Inventory and Order Tracking, Pricing, or Direct to Consumer Shipping (restricted) ([Amazon](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)).
- **How far back Amazon goes:** each request covers at most 30 days, "however, you can select any start date" ([Amazon](https://developer-docs.amazon/sp-api/docs/report-type-values-order#order-tracking-reports)). Orders more than two years old are only available through Archived Order Reports ([UK help](https://sellercentral.amazon.co.uk/help/hub/reference/external/GQE5U7B4QEZY5HEJ)).

## Columns

Tab-separated text, header row first. Columns as seen in a real UK seller's files (Oct 2026), in this order. Meanings are Amazon's where its docs give one.

| Column | Meaning | Format and traps |
|---|---|---|
| `amazon-order-id` | Amazon's unique, displayable identifier for an order. | Format `nnn-nnnnnnn-nnnnnnn`. One row per order item, so the id repeats for a multi-item order. |
| `merchant-order-id` | A unique identifier optionally supplied for the order by the seller. | Usually the same as `amazon-order-id`; blank on seller-fulfilled (`Merchant`) rows. |
| `purchase-date` | The date the order was placed. | ISO 8601 to the second, always `+00:00` (UTC), e.g. `YYYY-MM-DDThh:mm:ss+00:00`. The report period is chosen on this date. Business Reports (`sales_and_traffic`) counts days in UK time, so convert to UK time before comparing with it. |
| `last-updated-date` | The date of the most recent order update. | Same format. |
| `order-status` | Current status of the customer order. | Values include: `Shipped`, `Cancelled`, `Shipped - Delivered to Buyer`, `Pending`, `Shipping`, `Shipped - Waiting for Pick Up`. A file shows each order's status when you downloaded it, not today's. |
| `fulfillment-channel` | How the order was fulfilled. | `Amazon` or `Merchant`, not AFN/MFN. |
| `sales-channel` | Channel through which the order was purchased (the Amazon store). | e.g. `Amazon.co.uk`, with a capital A here (lower case in `fba_shipments`). Multi-Channel Fulfilment orders (FBA stock sent out for sales elsewhere) show as `Non-Amazon` or `Non-Amazon UK`. Leave those out of Amazon sales. |
| `order-channel` | The sub channel through which a sale was made for CBA/WBA orders. | `WebsiteOrderChannel` on seller-fulfilled (`Merchant`) rows; blank on FBA rows. |
| `ship-service-level` | The type of fulfilment service the buyer expects. | Values include: `Expedited`, `Standard`, `NextDay`. |
| `product-name` | The short title of the product. | |
| `sku` | The seller-defined product identifier. | Some sellers put their cost price in the SKU, so treat SKUs as private. |
| `asin` | Amazon inventory ID. | |
| `item-status` | Current status of this item within the order. | Values include: `Shipped`, `Cancelled`, `Unshipped`, blank. An item can be `Cancelled` in an order that wasn't. |
| `quantity` | The number of this item purchased. | **Usually `0` on cancelled lines**, including a `Cancelled` item inside an order that wasn't cancelled. Count rows, not `quantity`, for cancellations. |
| `currency` | The currency of the purchase. | ISO code, e.g. `GBP`. A file covering several stores can mix currencies. Usually blank on cancelled orders, and blank on Multi-Channel Fulfilment orders. |
| `item-price` | The amount the buyer paid for the item. | Total for the quantity, not a unit price. **Includes VAT**, unlike `item-price` in `fba_shipments`. Blank when `currency` is blank. |
| `item-tax` | The item tax the buyer paid. | Total for the quantity. **The VAT already inside `item-price`, not extra:** on a 20% VAT row it is within 1p of 1/6 of `item-price`. Never add it to `item-price`. With an item promotion it is the VAT after the promotion, so less than 1/6 (see `item-promotion-discount`). Can be blank, e.g. on exports with no VAT. |
| `shipping-price` | The amount the buyer paid for delivery. | Total for the quantity. Often blank, since FBA delivery is usually free. **Mixed VAT basis:** it is the net charge plus `shipping-tax`, and `shipping-tax` is the VAT after any delivery promotion. So it includes VAT when the buyer paid for delivery, but is net of VAT on free-delivery rows, where `shipping-tax` is blank. Either way, `shipping-price` minus `ship-promotion-discount` is what the buyer paid for delivery. That figure equals the item's `ShippingPrincipal`, `ShippingTax`, `ShippingDiscount` and `ShippingTaxDiscount` added up in `transactions` (each transaction counted once). |
| `shipping-tax` | The delivery tax the buyer paid. | |
| `gift-wrap-price` | The amount the buyer paid for gift wrap. | **Includes VAT**, like `item-price` and unlike `gift-wrap-price` in `fba_shipments`: it equals `transactions`' `GiftwrapPrincipal` plus `GiftwrapTax`, and `fba_shipments`' gift wrap price plus tax. |
| `gift-wrap-tax` | The gift wrap tax the buyer paid. | **The VAT already inside `gift-wrap-price`, not extra:** about 1/6 of it on a 20% row. Never add it to `gift-wrap-price`. |
| `item-promotion-discount` | Total of all promotion discounts applied to the order item. | **Positive** here (negative in `fba_shipments`). **Net of VAT**, while `item-price` includes VAT and `item-tax` is the VAT after the promotion. So the buyer paid `item-price` minus the discount with its VAT at the row's own rate: − 1.2 × `item-promotion-discount` on a 20% row (within 1p a unit of `transactions`), − 1.05 × on a 5% row; subtracting the discount alone overstates it by the discount's VAT. Where `item-tax` is `0` or blank (no VAT, e.g. zero-rated items or exports), subtract the discount as it is. In `transactions` the discount equals `OurPriceDiscount` (negative there), and `item-tax` equals `OurPriceTax` plus `OurPriceTaxDiscount`. |
| `ship-promotion-discount` | Promotion discount applied to delivery. | Positive (negative in `fba_shipments`): subtract it. Equals `shipping-price` on free-delivery rows. |
| `ship-city` | Delivery city. | Personal: never copy. |
| `ship-state` | Delivery state, region or county. | Personal: never copy. Often blank. |
| `ship-postal-code` | Delivery postcode. | Personal: never copy. |
| `ship-country` | Delivery country. | ISO 3166 two-letter code, e.g. `GB`. Can be blank. |
| `promotion-ids` | List of all item promotions applied to this order item. | Blank when no promotion applied. |
| `is-business-order` | Not defined in Amazon's docs; by name, an Amazon Business (B2B) order. | `true`/`false`. |
| `purchase-order-number` | Not defined in Amazon's docs; by name, the business buyer's PO number. | Can be blank. |
| `price-designation` | Not defined in Amazon's docs. | Can be blank on every row. |
| `is-iba` | Not in Amazon's docs. | `true`/`false`, sometimes `true`. Meaning not confirmed. |
| `order-item-id` | Not defined in Amazon's docs; Amazon's id for the item within the order. | 14 digits. Second-to-last column, not after `order-status` as Amazon's list has it. Matches `amazon-order-item-id` in `fba_shipments` and, for most items, `order-item-code` in `settlements` and `ORDER_ADJUSTMENT_ITEM_ID` in `transactions`. Some items there have an id in neither this report nor `fba_shipments`, though their order is here: join those on order id plus SKU. On Multi-Channel Fulfilment rows it's 1 character (`Non-Amazon`) or 24 to 27 characters (`Non-Amazon UK`); those rows never join. |
| `is-prime` | Not in Amazon's docs; by name, a Prime order. | `true`/`false`. Often `false` on FBA orders, so don't use it to count Prime orders. |

Documented but not in the file: `cpf` (a Brazil tax id). Amazon's help pages also list ship-from address columns, `tax-collection-model`, an IOSS number and Amazon programmes; none appear.

Source: [Order Reports: Flat File Orders By Order Date Report](https://developer-docs.amazon/sp-api/docs/report-type-values-order#order-tracking-reports); meanings from [All Orders Report (UK help)](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200547170) and [All Orders report (US help)](https://sellercentral.amazon.com/help/hub/reference/external/G200547170).

## Downloaded from Seller Central

**Nearly the same file as the API's, with one more column.** Seen in a real UK seller's Seller Central download (Oct 2026), for one day, by order date:

- **File name:** a number with `.txt`, not the period.
- **Format:** tab-separated, no byte order mark, lines ending in CRLF. `purchase-date` is ISO 8601 with an offset, as in Columns.
- **35 columns, not 34:** the columns above in the same order, plus **`url`** between `order-channel` and `ship-service-level`.
- **The last header is `is-prime ` with a trailing space.** Trim header names before matching on them, or `is-prime` won't be found.

The API file wasn't checked again for `url` or the trailing space, so a fresh API file may have them too. Read columns by trimmed name, never by position. The **Add or remove order report columns** link can also change the download's columns.

The download holds the same buyer location columns (see Personal data). The Order reports page has a banner saying buyers' personal details in order reports, phone numbers included, are for shipping labels only. For traps shared by all Seller Central downloads, see [seller-central-downloads](seller-central-downloads.md).

## Personal data

`ship-city`, `ship-state` and `ship-postal-code` are the buyer's location. Amazon says the report has no customer-identifying information ([Amazon](https://developer-docs.amazon/sp-api/docs/report-type-values-order#order-tracking-reports)), and there are no buyer names, emails or street addresses, but treat city, county and postcode as personal. Strip them before analysis and never copy them into anything shared.

## Questions it answers

- How many units of each ASIN or SKU sold, per store, per day or week? Leave out `Cancelled` rows and the Multi-Channel Fulfilment rows (`sales-channel` `Non-Amazon` or `Non-Amazon UK`).
- Which orders were cancelled, and how many per ASIN? Count rows: `quantity` is `0` on cancelled lines.
- Which store does an order belong to? (Join other reports' order ids to `sales-channel`.)
- How much did buyers pay per unit, per store, after promotions? (`item-price`, VAT included, minus the discount with its VAT at the row's own rate, such as 1.2 × `item-promotion-discount` on a 20% row, because the discount is net of VAT, or minus the discount alone when `item-tax` is `0` or blank; then divide by `quantity`, on rows that aren't cancelled.)
- Which promotions were applied to which order items?
- How fast does a new product sell through after it goes live?

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Cancelled orders stay in.** `Cancelled` rows usually have blank `currency` and `item-price` and `quantity` `0`. Leave them out of sales counts, and count them by row.
- **Seller-fulfilled orders are in too.** They show `fulfillment-channel` `Merchant` and `order-channel` `WebsiteOrderChannel`. They are Amazon sales, so keep them in units sold on Amazon and leave them out only when counting FBA.
- **Multi-Channel Fulfilment orders are in too.** Rows with `sales-channel` `Non-Amazon` or `Non-Amazon UK` are FBA stock sent out for sales elsewhere, with no price. Leave them out of units sold on Amazon.
- **Comparing with `transactions` at the start of a period.** This report picks orders by purchase date, but `transactions` is dated by when the money was posted, so a `transactions` file for a period holds lines for orders placed before it. No join reaches them: when comparing the two for the first weeks of the period you downloaded, leave out `transactions` lines whose `ORDER_ID` isn't in this report, or this report looks short.
- **`item-tax` and `gift-wrap-tax` are the VAT inside the price, not on top**, while `item-promotion-discount` is net of VAT (see Columns).
- Lines end in CRLF; the text is UTF-8.

Seen in Seller Central downloads (Oct 2026):

- **Check the date range covered.** Typing exact dates reset the end date, so the report covered one day only. Check the **Date Range Covered** column in the list of requests before using a download. The date boxes take MM/DD/YYYY.
- **A header with a trailing space:** `is-prime ` (see Downloaded from Seller Central).
