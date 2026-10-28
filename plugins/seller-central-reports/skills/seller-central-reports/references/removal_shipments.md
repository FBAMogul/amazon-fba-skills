# Removal Shipment Detail (`removal_shipments`)

Shipments sent out for FBA removal orders, one row per item shipment, with carrier and tracking number; Amazon says it leaves out cancelled and disposed items, which are in [`removal_orders`](removal_orders.md), though disposed units do turn up here ([Removal Shipment Detail](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200989100); see Traps seen in real data).

## Where to find it in Seller Central

**Supply chain > Supply chain reports** (the Amazon Fulfilment Reports page), then **Removals > Removal Shipment Detail** in the left-hand list; it is also linked as **Removal Delivery Detail**. Classic: **Reports > Fulfilment by Amazon**. Checked in Seller Central UK on 28 Oct 2026. Amazon's help groups it with the removal reports ([Removal Reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200989090)) and links it at `sellercentral.amazon.co.uk/reportcentral/REMOVAL_SHIPMENT_DETAIL/1` ([Amazon Fulfilment reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201104750)).

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_FBA_FULFILLMENT_REMOVAL_SHIPMENT_DETAIL_DATA`, `marketplaceIds` set to every store the account takes part in (for a UK account, its stores in Amazon's EU region), and `dataStartTime`/`dataEndTime` set to the window; then poll `getReport` until it's done and fetch the file with `getReportDocument` (unzip it if Amazon says GZIP). No report options. Amazon updates it in near real time ([FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). The window selects rows by `request-date`, not `shipment-date`, and Amazon can throttle `createReport` for this report type (see Traps seen in real data).
- **Role:** [Amazon Fulfillment or Pricing](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types) (either is enough).
- **How far back Amazon goes:** not stated in Amazon's docs.

## Columns

Tab-separated text, header row first, 10 columns in this order.

| Column | Meaning | Format and traps |
|---|---|---|
| `request-date` | Date the removal order was submitted | ISO 8601; the time zone offset varies from row to row. Often not the offset [`removal_orders`](removal_orders.md) gives the same order (the same moment, written differently), so compare in UTC. |
| `order-id` | Removal order ID, system-generated or set by the seller | 9 or 10 characters: letters and digits, sometimes with `/` or `+`. Matches `order-id` in [`removal_orders`](removal_orders.md). |
| `shipment-date` | Date the fulfilment centre shipped it | Offset varies, with the same values as `request-date`. Can be weeks or months after `request-date`. |
| `sku` | Seller's SKU | **HTML entities:** `£` comes as `&pound;`, `"` as `&quot;`. Decode every entity before joining; replacing `&pound;` alone still leaves some SKUs that won't match. |
| `fnsku` | Amazon's fulfilment network SKU | |
| `disposition` | Status of the unit | `Unsellable` or `Sellable`. |
| `shipped-quantity` | Units of this product shipped for this return | |
| `carrier` | Carrier that delivered the parcel | Amazon's carrier codes, such as `HERMES_UK_PARCEL_48`, `UPS_SMALLPACK_UK` and `ROYAL_TRACKED_48`; others occur. Blank on `Disposal` rows. |
| `tracking-number` | Carrier's tracking number, if available | Blank on `Disposal` rows. |
| `removal-order-type` | Not defined in Amazon's docs | `Return` or `Disposal`. |

Documented but not in the file: `shipment-status` (India only).

## Personal data

None of the columns hold buyer or business personal details, and the report has no address columns. Some sellers put their cost price in the SKU, so treat SKUs as private.

## Questions it answers

- Which carrier and tracking number does each removal shipment have, to chase a delivery?
- Did every returned unit that wasn't cancelled ship? For return lines, match [`removal_orders`](removal_orders.md) on `order-id` and `fnsku`, and compare with its `requested-quantity` minus `cancelled-quantity` minus `disposed-quantity` (blank is 0).
  - Not with `requested-quantity` alone: cancelled units never ship, so they look unshipped, and return lines can have some.
  - Not with its `shipped-quantity` either: the two reports, fetched at the same time, agree with each other even when units haven't shipped yet.
  - A shipment made after the copy was fetched won't be in it, so a gap may mean the copy is out of date, not that nothing shipped.
- How many units shipped out for removals each month? Group the `Return` rows by `shipment-date` (in UTC), not by the month the report was requested for: a report for one month holds that month's requests, and their shipments can fall in later months. Leave out the `Disposal` rows: they were disposed, not shipped. Don't expect it to balance with the ledger's `VendorReturns` in [`ledger_detail`](ledger_detail.md), which counts those disposed units too, on the same FNSKU and day: even with the `Disposal` rows kept, monthly totals often disagree, and some ledger removal rows are for FNSKUs in neither removal report.
- Which removal shipments to cite when a delivery arrives short?

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Throttled when many months were requested.** After about a dozen quick requests (one per month), Amazon refused `createReport` for this report type with HTTP 429 for the rest of that run, about 40 minutes of retries, while still accepting other report types. A later run fetched the rest. Amazon doesn't document this.
- **Selected by request month, not shipment month.** Every row sits in the report window for its `request-date`. Rows can ship in a later month, weeks or months after the request month ended. Fetch recent months again, or a late shipment will be missing.
- **`Disposal` rows can appear**, although Amazon says this report leaves out disposed items. They belong to disposal lines in [`removal_orders`](removal_orders.md), which show every unit disposed and `shipped-quantity` blank, and they're the only rows with a blank `carrier` and `tracking-number`. Leave them out of the shipped check and of units shipped out (above), but keep them when matching `VendorReturns` in [`ledger_detail`](ledger_detail.md), which has them.
- **Date offsets vary by row**, and differ from [`removal_orders`](removal_orders.md) for the same order. Convert to UTC.
- Lines end in CRLF, but the last line of every file has no line ending at all, so `wc -l` counts one row short. The text is UTF-8.

Sources: [FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba), [Removal Shipment Detail](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200989100).
