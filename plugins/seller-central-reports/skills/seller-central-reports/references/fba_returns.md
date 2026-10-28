# FBA customer returns (`fba_returns`)

Customer-returned FBA items received at an Amazon fulfilment centre, one row per returned item line, with return reason, disposition and status; it holds FBA returns only (seller-fulfilled returns are in [`fbm_returns`](fbm_returns.md)), and a refund whose item never reached a fulfilment centre isn't here ([FBA customer returns report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200453320)).

## Where to find it in Seller Central

**Supply chain > Supply chain reports** (the Amazon Fulfilment Reports page), then **Customer Concessions > FBA customer returns** in the left-hand list. **Orders > Order reports** also has an **FBA customer return reports** tab, which opens the same report (`/reportcentral/CUSTOMER_RETURNS/0`). Classic: **Reports > Fulfilment by Amazon**. Checked in Seller Central UK on 28 Oct 2026. Amazon's help groups it with its customer concessions reports ([Customer Concessions reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200885410)) and links the same address ([FBA customer returns report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200453320)).

A wider view, from return authorisation to receipt, is the FBA Returns page. Amazon's help reaches it from the FBA Dashboard (classic: **Inventory > FBA Dashboard**), then the **Inventory** tab > **FBA Returns** ([FBA Returns page](https://sellercentral.amazon.co.uk/help/hub/reference/external/G202191620)). The new menu's **Site map** lists **Manage FBA returns** (`/fba-return`) under Orders; it wasn't opened, so whether it is the same page is not confirmed.

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_FBA_FULFILLMENT_CUSTOMER_RETURNS_DATA`, `marketplaceIds` set to every store the account takes part in (for a UK account, its stores in Amazon's EU region), and `dataStartTime`/`dataEndTime` set to the window; then poll `getReport` until it's done and fetch the file with `getReportDocument` (unzip it if Amazon says GZIP). No report options. Amazon updates it daily ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)).
- **Role:** [Amazon Fulfillment or Pricing](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types) (either is enough).
- **How far back Amazon goes:** not stated in Amazon's docs.

## Columns

Tab-separated text, header row first, 13 columns in this order.

| Column | Meaning | Format and traps |
|---|---|---|
| `return-date` | Date the returned item was processed at the fulfilment centre | ISO 8601 with offset (`YYYY-MM-DDThh:mm:ss+00:00`), not the help page's DD-MM-YYYY. |
| `order-id` | Amazon order ID of the original shipment, if available | |
| `sku` | Seller's SKU | **HTML entities:** `£` comes as `&pound;`; `&amp;` and `&quot;` occur too. Decode every entity before joining. |
| `asin` | Amazon product identifier | |
| `fnsku` | Amazon's fulfilment network SKU | |
| `product-name` | Product name as it appears on Amazon | Can be cut short. |
| `quantity` | Units of this item returned | |
| `fulfillment-center-id` | Fulfilment centre that processed the return | Amazon's fulfilment centre code. |
| `detailed-disposition` | Condition of the returned item | Codes such as `SELLABLE`, `CUSTOMER_DAMAGED` and `CARRIER_DAMAGED`, with underscores, not the help page's spaces. |
| `reason` | Customer's return reason | A code, such as `UNWANTED_ITEM`, `DAMAGED_BY_FC` or `DAMAGED_BY_CARRIER`; there are many. |
| `status` | Status of the return | Text such as `Unit returned to inventory`, `Reimbursed` or `Repackaged Successfully`. `Unit returned to inventory` doesn't mean sellable: check `detailed-disposition`. |
| `license-plate-number` | Serial number tracking the specific item through fulfilment | Starts `LPN`. |
| `customer-comments` | Customer's comments on the reason, when available | Free text in the buyer's own words; often blank. Personal data (see below). |

## Personal data

- `customer-comments`: free text written by buyers, which can hold anything they chose to type.

Strip it before analysis and never copy it into anything shared. So Claude doesn't read or summarise what buyers wrote: use `reason` and `detailed-disposition` for why items came back, and the seller can read the comments themselves in Seller Central. Some sellers put their cost price in the SKU, so treat SKUs as private too.

## Questions it answers

- Which ASINs are returned most, and for what reasons?
- How many returned units went back to sellable stock, and how many came back unsellable? Go by `detailed-disposition`, not `status`, except for `Repackaged Successfully`: those rows can say `CUSTOMER_DAMAGED`, yet the same FNSKU turns sellable in [`ledger_detail`](ledger_detail.md) (a `P` adjustment) shortly after. Check the ledger for those.
- Which returns were damaged by Amazon or its carrier (`detailed-disposition` `CARRIER_DAMAGED`, or a `reason` such as `DAMAGED_BY_CARRIER` or `DAMAGED_BY_FC`), and does each have a matching line in [`reimbursements`](reimbursements.md)?
- Which refunded orders (from [`transactions`](transactions.md) or [`settlements`](settlements.md)) have no return here, so may need a claim?

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Codes use underscores** (`CUSTOMER_DAMAGED`, `CARRIER_DAMAGED`), unlike the help page.
- **Returned to inventory isn't sellable.** `Unit returned to inventory` rows can be customer damaged or defective; only the `SELLABLE` ones came back sellable.
- **`Repackaged Successfully` rows can say `CUSTOMER_DAMAGED`**, though the units can later turn sellable in the ledger (see Questions it answers).
- **`CARRIER_DAMAGED` rows can have `status` `Reimbursed`.**
- **`customer-comments` is often blank.**
- **HTML entities in SKUs:** `&pound;`, `&amp;` and `&quot;`. Replacing `&pound;` alone leaves some SKUs unmatched against the plain-text reports ([`orders`](orders.md), [`listings_all`](listings_all.md), [`inventory_all`](inventory_all.md), [`ledger_detail`](ledger_detail.md), [`fba_shipments`](fba_shipments.md)); after decoding every entity, all match.
- **Not throttled:** Amazon accepted many months requested one after another for this report.
- Lines end in CRLF, but the last line of each file has no line ending at all, so `wc -l` counts one row short and a shell `while read` loop skips that row. The text is UTF-8.

Sources: [FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba), [FBA customer returns report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200453320).
