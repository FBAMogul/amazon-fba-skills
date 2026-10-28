# Amazon-Fulfilled Shipments report (`fba_shipments`)

Every completed FBA shipment to a buyer, item by item, with prices, carrier, tracking and fulfilment centre: Amazon's FBA Amazon Fulfilled Shipments Report, "detailed order/shipment/item information including price, courier, and tracking data" ([Amazon](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)), listing shipments as they are reported to FBA in the period.

## Where to find it in Seller Central

**Supply chain > Supply chain reports** (the Amazon Fulfilment Reports page), then **Sales > Amazon Fulfilled Shipments** in the left-hand list (`/reportcentral/AFNShipmentReport/1`). **Amazon Fulfilled Shipments - Tax Invoicing**, next to it, is the Tax Invoicing version, which the [UK help](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200453120) covers on the same page. Classic: **Reports > Fulfilment by Amazon**. Checked in Seller Central UK on 28 Oct 2026.

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_AMAZON_FULFILLED_SHIPMENTS_DATA_GENERAL`, the marketplace ids of the stores you want, and `dataStartTime`/`dataEndTime` set to the period; then `getReport` until the report is done, and `getReportDocument` (unzip it if Amazon says GZIP). No report options.
- **Role:** any one of Amazon Fulfillment, Inventory and Order Tracking or Pricing ([Amazon](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)).
- **How far back Amazon goes:** shipments up to 18 months back, at most 31 days per request ([UK help](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200453120)).
- **Rate limit:** Amazon accepts at most 12 requests for this report every 8 hours per account, and Seller Central downloads count towards the 12 ([Amazon](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). Amazon doesn't accept requests past the limit until earlier ones fall outside the 8 hours, so a refused or cancelled request here can mean you hit it: wait and ask again.
- **Delay:** items usually appear one to three hours after shipping, rarely up to 24 hours ([Amazon](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). Amazon's help says an item shipped late in a period can land in the next one ([UK help](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200453120)).

## Columns

Tab-separated text, header row first. Columns as seen in a real UK seller's files (Oct 2026), in this order.

| Column | Meaning | Format and traps |
|---|---|---|
| `amazon-order-id` | Amazon's unique, displayable identifier for an order. | |
| `merchant-order-id` | A unique identifier optionally supplied for the order by the merchant. | Usually blank. |
| `shipment-id` | Encrypted Amazon shipment ID. | 9 letters and digits. |
| `shipment-item-id` | Encrypted Amazon shipment item ID. | Unique per row. 13 letters and digits, sometimes 9. |
| `amazon-order-item-id` | Amazon's unique, displayable identifier for an item within the order. | 14 digits. Same as `order-item-id` in `orders`. **Not unique here:** one item can have several rows, each with its own `shipment-item-id`, usually all in one shipment, sometimes across more than one `shipment-id`. An item's rows add up to its `orders` line. So joining `orders` to this report and summing `orders`' price or quantity per joined row counts those items more than once. |
| `merchant-order-item-id` | A unique identifier for an item within the order. | Usually blank. |
| `purchase-date` | The date the order was placed. | ISO 8601 with offset, e.g. `YYYY-MM-DDThh:mm:ss+00:00`. |
| `payments-date` | The date the buyer's card was charged and order processing was completed. | Same format. |
| `shipment-date` | The date the fulfilment centre completed (dispatched) the shipment. | Same format. Based on when the shipment was reported to the system. |
| `reporting-date` | The date the data became available in the FBA data store. | Same format. |
| `buyer-email` | The buyer's email address. | Personal: never copy. Usually filled. |
| `buyer-name` | The buyer's full name. | Always blank. |
| `buyer-phone-number` | The buyer's phone number. | Always blank. |
| `sku` | The merchant-defined product identifier. | |
| `product-name` | The short title of the product. | |
| `quantity-shipped` | The number of this item dispatched. | |
| `currency` | The currency of the purchase. | ISO code, e.g. `GBP`. A file covering several stores can mix currencies. |
| `item-price` | The item's price: net of VAT for a seller enrolled in Amazon's VAT Calculation Service, VAT-inclusive otherwise ([Amazon](https://developer-docs.amazon/sp-api/docs/create-and-upload-invoices)). | Total for the quantity, not a unit price. **Check which:** where it's net of VAT, `item-price` + `item-tax` equals `orders`' `item-price` (which includes VAT) for an item with one row here. Blank only on `Non-Amazon` (Multi-Channel Fulfilment) rows, which have every money column blank. |
| `item-tax` | The VAT on the item, on top of `item-price` when that's net of VAT. | Total for the quantity. |
| `shipping-price` | The amount the buyer paid for shipping. | Net of VAT on the same basis as `item-price`. Zeros are written as `0.00`, not blank (blank only on `Non-Amazon` rows). |
| `shipping-tax` | The shipping tax the buyer paid. | `0.00` when none. |
| `gift-wrap-price` | The amount the buyer paid for gift wrap. | `0.00` when none. Net of VAT on the same basis as `item-price`, unlike `orders`' `gift-wrap-price`: this plus `gift-wrap-tax` equals the `orders` figure. |
| `gift-wrap-tax` | The gift wrap tax the buyer paid. | `0.00` when none. |
| `ship-service-level` | The type of fulfilment service the buyer expects. | Values include `Expedited`, `Standard` and `premium-uk-nominated-day`. |
| `recipient-name` | The name on the delivery address. | Always blank. |
| `ship-address-1`, `ship-address-2`, `ship-address-3` | Delivery address lines. | Always blank. |
| `ship-city` | Delivery city. | Personal: never copy. Blank where `ship-country` is `--`, and can be blank on orders shipped outside the UK and EU. |
| `ship-state` | Delivery state, region or county. | Personal: never copy. Blank where `ship-country` is `--`. |
| `ship-postal-code` | Delivery postcode. | Personal: never copy. Blank where `ship-country` is `--`. |
| `ship-country` | Delivery country. | ISO 3166 two-letter code, e.g. `GB`; can be `--`. |
| `ship-phone-number` | Phone number for the delivery address. | Always blank. |
| `bill-address-1` to `bill-address-3`, `bill-city`, `bill-state`, `bill-postal-code`, `bill-country` | Billing address. | Always blank. |
| `item-promotion-discount` | Promotion discount applied to the order item. | `0.00` or **negative** (positive in `orders`): add it. |
| `ship-promotion-discount` | Promotion discount applied to delivery. | `0.00` or **negative** (positive in `orders`). It usually cancels the net `shipping-price` exactly (free delivery) but not `shipping-tax`. Sometimes it is smaller than `shipping-price`. |
| `carrier` | The courier the parcel was dispatched with. | Amazon's carrier code, e.g. `AMZN_UK`, `ROYAL_MAIL`, `DPD`, `APEX`. Can be blank. |
| `tracking-number` | The courier's tracking number. | |
| `estimated-arrival-date` | The estimated delivery date. | ISO 8601 with offset. |
| `fulfillment-center-id` | The fulfilment centre the order was dispatched from. | Amazon's fulfilment-centre code. |
| `fulfillment-channel` | How the order was fulfilled. | Always `AFN` (unlike `Amazon` in `orders`). |
| `sales-channel` | The source of the order (the Amazon store). | Lower case, e.g. `amazon.co.uk`, unlike `orders`. `Non-Amazon` on Multi-Channel Fulfilment rows. |

Documented but not in the file: `points-granted` (JP only).

Source: [FBA Reports: FBA Amazon Fulfilled Shipments Report](https://developer-docs.amazon/sp-api/docs/report-type-values-fba); meanings from [Amazon-Fulfilled Shipments Report (UK help)](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200453120).

## Personal data

`buyer-email` (usually filled), and `ship-city`, `ship-state` and `ship-postal-code` (the buyer's location). `buyer-name`, `buyer-phone-number`, `recipient-name`, `ship-phone-number` and the address and billing columns were always blank in the files seen, but treat them as personal if they ever fill. Strip them before analysis and never copy them into anything shared.

## Questions it answers

- Which fulfilment centre ships each SKU, and to which countries?
- How many units shipped per SKU per store per month?
- What did the buyer pay per shipped item, including delivery and promotions? Where `item-price` is net of VAT (see the column), add `item-price` and `item-tax`, delivery and gift wrap with their tax, and the two promotion discounts (negative here). When `ship-promotion-discount` cancels `shipping-price` (free delivery), leave out `shipping-tax` too: `settlements` shows that VAT cancelled by a `TaxDiscount` promotion row. When the promotion is smaller than `shipping-price`, take the promotion's VAT off `shipping-tax` instead, at the row's own rate (`shipping-tax` ÷ `shipping-price`): `transactions` shows a `ShippingTaxDiscount` of that size. Never assume 20%: the report can cover stores in other currencies, items at other VAT rates, and items delivered abroad whose VAT isn't 20% (see `settlements`). 20% is only the UK standard rate: there, a £1.00 promotion takes £0.20 off the tax. An item promotion on a row with `item-tax` `0.00` (no VAT, e.g. an export) needs no VAT step: adding the discount is exact. On a taxed item, check whether `item-tax` is before the promotion, as `shipping-tax` is; if it is, take the promotion's VAT off `item-tax` too, at the row's own rate (`item-tax` ÷ `item-price`). In `orders` the item promotion is net of VAT.
- How long from purchase to dispatch for FBA orders?
- Which shipments went to which store's buyers (via `sales-channel`), for cross-border sales from UK stock?

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Almost the same rows as `fba_shipment_sales`.** Month by month the row counts match within a few rows. Use this report for order-level detail (order item id, carrier, tracking, dates); use `fba_shipment_sales` for the unit price and FNSKU. Item promotions are only in this report and `orders`.
- **One item can have several rows** (see `amazon-order-item-id`), so never sum `orders`' price or quantity per joined row.
- **Shipments stayed in their month.** Despite Amazon's help, every `shipment-date` fell inside the month requested.
- **Buyer email is usually filled; name, phone and address lines are always blank.** City, county and postcode are filled except on rows whose `ship-country` is `--`, where all three are blank, and city can also be blank on orders shipped outside the UK and EU.
- **Plan around the 12-requests-per-8-hours limit.** Pulling 18 months a month at a time takes two sittings at least 8 hours apart: 12 months, then the rest. Before saving any copy to keep, drop `buyer-email` and the other personal columns (see Personal data): Amazon's [Data Protection Policy](https://sellercentral.amazon.com/mws/static/policy?documentType=DPP&locale=en_GB) says data from the API that identifies buyers is kept "no longer than 30 days after order delivery" unless a law requires it.
- Lines end in CRLF, but the last line of every file has no line ending at all, so counting lines gives one fewer than the rows. The text is UTF-8.
