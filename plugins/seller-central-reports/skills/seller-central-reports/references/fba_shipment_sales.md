# FBA Customer Shipment Sales report (`fba_shipment_sales`)

Shipped FBA customer orders, one row per item shipped, with unit price, FNSKU, fulfilment centre and ship-to location: Amazon's FBA Customer Shipment Sales Report, "condensed item level data on shipped FBA customer orders including price, quantity, and ship-to location" ([Amazon](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)), which Amazon suggests for reconciling stock because it gives the shipped quantity of a SKU in a period ([UK help](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201104750)).

## Where to find it in Seller Central

**Supply chain > Supply chain reports** (the Amazon Fulfilment Reports page; [Amazon Fulfilment reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201104750)), then **Sales > Customer Delivery Sales** in the left-hand list (`/reportcentral/SHIPMENT_SALES/0`, which opens the online view). Amazon now labels it Customer Delivery Sales, not FBA Customer Shipment Sales. Classic: **Reports > Fulfilment by Amazon**. Checked in Seller Central UK on 28 Oct 2026.

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_FBA_FULFILLMENT_CUSTOMER_SHIPMENT_SALES_DATA`, the marketplace ids of the stores you want, and `dataStartTime`/`dataEndTime` set to the period; then `getReport` until the report is done, and `getReportDocument` (unzip it if Amazon says GZIP). No report options.
- **Role:** Amazon Fulfillment ([Amazon](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)).
- **How far back Amazon goes:** not stated on the SP-API page. The US help page says "Customer order data is kept for 18 months" ([US help](https://sellercentral.amazon.com/help/hub/reference/external/G200453140)); the UK help page for this report returned no text when checked.
- **Delay:** items usually appear one to three hours after shipping, rarely up to 24 hours ([Amazon](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)).

## Columns

Tab-separated text, header row first. Columns as seen in a real UK seller's files (Oct 2026), in this order.

| Column | Meaning | Format and traps |
|---|---|---|
| `shipment-date` | Date the order was shipped to the customer. | ISO 8601 with offset, e.g. `YYYY-MM-DDThh:mm:ss+00:00`. |
| `sku` | Product identifier assigned by the seller. | |
| `fnsku` | Amazon Fulfillment Network product identifier. | Usually starts `X00`. For stock tracked by the manufacturer barcode it can be the ASIN ([Amazon's reply in the seller forums](https://sellercentral.amazon.in/seller-forums/discussions/t/00d44f01-bf57-4215-9a18-8660ec840c0f)). |
| `asin` | Product identifier assigned by Amazon. | |
| `fulfillment-center-id` | The fulfilment centre the order was shipped from. | Amazon's fulfilment-centre code. |
| `quantity` | The number of product units ordered. | Often `1`; multi-unit lines happen. |
| `amazon-order-id` | Amazon's identifier for the order. | |
| `currency` | Local currency for the transaction. | ISO code, e.g. `GBP` or `EUR`. |
| `item-price-per-unit` | Price of each product unit: net of VAT for a seller enrolled in Amazon's VAT Calculation Service, as `fba_shipments`' `item-price` is (see that page). | A unit price, unlike `item-price` in `orders` and `fba_shipments`. **Before item promotions:** this report has no item-promotion column, so on promoted lines it is the price before the discount (times `quantity`, it equals `fba_shipments`' `item-price`, which is also before its `item-promotion-discount`). Take item promotions from `fba_shipments` or `orders`. **Same basis as `fba_shipments`:** times `quantity` it equals `fba_shipments`' `item-price`; `orders`' `item-price` includes VAT. On a multi-unit line whose total doesn't divide evenly by `quantity`, the unit price is rounded to the penny, so times `quantity` it can be a penny out. |
| `shipping-price` | The shipping charge for each product unit. | **Per line, not per unit**, despite Amazon's wording; for a VAT Calculation Service seller, net of VAT and before any delivery promotion. Where it isn't zero it equals `fba_shipments`' `shipping-price` for the line. A delivery promotion, which this report doesn't carry, can cancel the charge, so it isn't always what the buyer paid. `0.00` when none. |
| `gift-wrap-price` | Charge for gift wrapping, if the customer asked for it. | `0.00` when none. |
| `ship-city` | Destination city. | Personal: never copy. |
| `ship-state` | Destination state, region or province. | Personal: never copy. |
| `ship-postal-code` | Destination postcode. | Personal: never copy. |

Documented but not in the file: `points-granted` (JP only). There is no store column; the `currency` hints at the store, and joining `amazon-order-id` to `orders` gives `sales-channel`.

Source: [FBA Reports: FBA Customer Shipment Sales Report](https://developer-docs.amazon/sp-api/docs/report-type-values-fba); meanings from [Customer Shipment Sales Report (US help)](https://sellercentral.amazon.com/help/hub/reference/external/G200453140).

## Personal data

`ship-city`, `ship-state` and `ship-postal-code` are the buyer's location. Strip them before analysis and never copy them into anything shared.

## Questions it answers

- How many units of each SKU or FNSKU shipped per month, to reconcile against the inventory ledger?
- What unit price did each shipped item sell at (net of VAT for a VAT Calculation Service seller)? (Before item promotions. For the unit price after them, use the matching `fba_shipments` line: (`item-price` + `item-promotion-discount`) ÷ `quantity-shipped`, as both are totals for the line and the discount is negative there; don't take the whole line's discount off one unit's price.)
- Which fulfilment centres ship each SKU?
- How many units of each SKU shipped to each destination country? This report has no country column: take each order's `ship-country` from `fba_shipments` or `orders` (joined on `amazon-order-id`, one country per order) and add up `quantity` here. No buyer columns are needed.

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Same shipments as `fba_shipments`:** the monthly row counts match it within a few rows. This report adds the FNSKU and a per-unit price; `fba_shipments` adds the order item id, item promotions, carrier and tracking.
- **`CONFIDENTIAL` in the location columns.** On orders whose `ship-country` is `--` in `fba_shipments`, city, county and postcode all say `CONFIDENTIAL`. `ship-city` can also be blank on other orders.
- **Not throttled.** Pulling many months, a month per request, in one go was never throttled, unlike several other FBA reports.
- Lines end in CRLF, but the last line of every file has no line ending at all, so counting lines gives one fewer than the rows. The text is UTF-8.
