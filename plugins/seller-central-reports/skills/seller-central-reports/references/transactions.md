# Transactions (Finances API) (`transactions`)

Every financial transaction Amazon posted to the account (order payments, refunds, fees, transfers), with its fee breakdown and its deferred or released status.

## Where to find it in Seller Central

Checked in Seller Central UK on 28 Oct 2026: **Finance > Finance reports**, which opens **Payment reports**, then the **Transaction View** tab. The other tabs there include All Statements (the [`settlements`](settlements.md) files) and Reports Repository, which has the date range transaction report ([`date_range_transactions`](date_range_transactions.md)). Filter by account type, transaction type, status and period (a custom range in DD/MM/YYYY, a number of past days, or a settlement period). Each row shows whether it is deferred or released.

Amazon's help lists Transaction View among the Payments reports ([Payments reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200885400)), and also describes a **Manage Transaction** view opened from the Balance Summary card ([Transaction View](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200378780)).

## Getting it through the API

- **Call / report type:** Finances API 2024-06-19 `listTransactions` with `postedAfter` and `postedBefore` set to the window. Page with `nextToken`, sending the same `postedAfter` and `postedBefore` on every page, as Amazon asks ([listTransactions](https://developer-docs.amazon/sp-api/reference/listtransactions)). The two times must be no more than 180 days apart (a longer window comes back empty) and both more than two minutes before the request ([listTransactions](https://developer-docs.amazon/sp-api/reference/listtransactions)). With no `transactionStatus` filter, the response includes `RELEASED`, `DEFERRED` and `DEFERRED_RELEASED` transactions ([Get the latest transactions](https://developer-docs.amazon/sp-api/docs/get-latest-transactions)); with no `marketplaceId`, every store's transactions come back together. Orders from the last 48 hours may be missing ([listTransactions](https://developer-docs.amazon/sp-api/reference/listtransactions)). Not a report: the response is JSON, one object per transaction in `payload.transactions`.
- **Role:** [Finance and Accounting](https://developer-docs.amazon/sp-api/docs/role-mappings).
- **How far back Amazon goes:** not stated in Amazon's API docs. Seller Central's Transaction View shows only the last 24 months ([Transaction View](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200378780)).

## Columns

One transaction per JSON object. Each transaction and item has a `breakdowns` list showing how its total is built; `breakdownType` is an open list that Amazon adds to as needed ([Get the latest transactions](https://developer-docs.amazon/sp-api/docs/get-latest-transactions)).

Money fields are objects with `currencyCode` and `currencyAmount` (a signed decimal), not `amount`. **Not all in pounds:** `currencyCode` is the store's currency, the same on every amount in a transaction: `GBP` for the UK, `EUR` for the euro stores, and the local currency for others, such as `SEK` for Sweden. The exception is an item with an empty `breakdowns` list, whose zero `totalAmount` has `currencyCode` `null`. Total each currency separately; never add EUR or SEK amounts to GBP ones. `contexts` and `breakdowns` have the same shape at transaction level and item level, so they're described once, but item-level breakdowns skip the `Sales` and `Expenses` level (see `breakdownType`). `contexts`, `items[].contexts` and `items[].relatedIdentifiers` can be `null` instead of an empty list. `breakdowns` and `items[].breakdowns` never are (though they can be an empty list), but a child's own `breakdowns` ends as `null`, an empty list or no field at all.

| Column | Meaning | Format and traps |
|---|---|---|
| `sellingPartnerMetadata.sellingPartnerId` | The seller's unique id | The same on every transaction. |
| `sellingPartnerMetadata.accountType` | Type of account in the transaction | e.g. `Standard Orders`. |
| `sellingPartnerMetadata.marketplaceId` | Store where the transaction happened | Amazon's store id, e.g. `A1F83G8C2ARO7P` for the UK. |
| `relatedIdentifiers[].relatedIdentifierName` | Kind of linked id | Values include: `FINANCIAL_EVENT_GROUP_ID` and `SETTLEMENT_ID` (on every transaction), `ORDER_ID`, `SHIPMENT_ID`, `OBFUSCATED_SHIPMENT_ID` (not in Amazon's list), `RELEASE_TRANSACTION_ID`, `DEFERRED_TRANSACTION_ID`, `REFUND_ID`. |
| `relatedIdentifiers[].relatedIdentifierValue` | The linked id's value | `SETTLEMENT_ID` matches `settlement-id` in [`settlements`](settlements.md). |
| `transactionType` | Type of transaction | Amazon's model names only `Shipment`. Others include `Refund`, `ServiceFee`, `Adjustment`, `Transfer`, `FBAInventoryReimbursement` and `ProductAdsPayment`. |
| `transactionId` | Unique id of the transaction | Seen as 43 characters. |
| `transactionStatus` | `DEFERRED`, `RELEASED`, or `DEFERRED_RELEASED` (deferred earlier, now released) | Changes from `DEFERRED` to `DEFERRED_RELEASED` on release. The status is as of when you fetched it: a copy fetched before the release still says `DEFERRED`, so fetch recent weeks again before relying on it. |
| `description` | Reason for the transaction | Many values. Amazon's examples are `Order Payment` and `Refund Order`; others include `Refund`, `Disbursement`, `FBAStorageBilling`, `FBALongTermStorageBilling`, `FBARemoval` and `FBADisposal`. |
| `postedDate` | When the transaction was posted | ISO 8601, UTC (`YYYY-MM-DDThh:mm:ssZ`). |
| `totalAmount` | Total money in the transaction | Money object, in the store's currency (see above). |
| `marketplaceDetails.marketplaceId` | Store where the transaction happened | Amazon's store id. |
| `marketplaceDetails.marketplaceName` | Store name | e.g. `Amazon.co.uk`; can also be a `Non-Amazon …` name, such as `Non-Amazon UK`. |
| `items[].description` | Description of the items in the transaction | The product title. |
| `items[].relatedIdentifiers[].itemRelatedIdentifierName` | Kind of item id | Values include: `ORDER_ADJUSTMENT_ITEM_ID`, `TRANSACTION_ID`, `SETTLEMENT_ID` and `INVOICE_ID` (the last two not in Amazon's item list). |
| `items[].relatedIdentifiers[].itemRelatedIdentifierValue` | The item id's value | 14 digits for `ORDER_ADJUSTMENT_ITEM_ID`, which matches `order-item-id` in [`orders`](orders.md) for most items, **but not all.** Some `Shipment` and `Refund` items have an id that is in neither `orders` nor [`fba_shipments`](fba_shipments.md), for two reasons. Some belong to orders outside the period your `orders` data covers (for example where `orders` starts later than `transactions`): no join reaches those, so leave them out when comparing the two. Others have an `ORDER_ID` that is in `orders`: joining earnings to `orders` by item id drops those sales, so fall back to `ORDER_ID` plus `contexts[].sku`. Drop the `orders` rows whose `item-status` is `Cancelled` first: a cancelled row (quantity 0) can sit beside the `Shipped` row for the same SKU. If an order has two `Shipped` rows for the SKU, match `quantityShipped` to `quantity`. Rarely, the `orders` row has a different SKU for the same ASIN: match on order id plus ASIN. A row can match several items, though: their `quantityShipped` adds up to the row's `quantity`, and their `OurPricePrincipal` plus `OurPriceTax` to its `item-price`. So add up `transactions` per `orders` row and never sum `orders`' price or quantity per joined item. |
| `items[].totalAmount` | Total money for the item | Money object. |
| `items[].breakdowns[]` | How the item's total is calculated | Same shape as `breakdowns[]`. |
| `items[].contexts[]` | Extra information about the item | Same shape as `contexts[]`. |
| `breakdowns[].breakdownType` | Type of charge | At transaction level the top is `Sales` and `Expenses` (or `Refunded Sales` and `Refunded Expenses`) on every transaction, with `ProductCharges`, `Tax`, `Shipping`, `AmazonFees`, `DigitalServicesFee`, `PromoRebates`, `FBAFees`, `StorageBillingFee`, `AdvertisingFee` and more below them. Item-level breakdowns have no such level: they start at the charge types, such as `ProductCharges`, `Tax` and `AmazonFees`. Where Amazon calculates the VAT (its VAT Calculation Service), `ProductCharges` is the price net of VAT and the `Tax` beside it is the VAT (20% of `ProductCharges`, within a penny, on a standard-rated UK `Shipment`). With no `Tax` breakdown, don't treat `ProductCharges` as net: ask the seller and use their accounting records (see the VAT section of [SKILL.md](../SKILL.md)). Where Amazon keeps the VAT, a `MarketplaceFacilitatorVAT-…` line under `Tax` cancels it (see the traps). |
| `breakdowns[].breakdownAmount` | Amount of that charge | Money object. |
| `breakdowns[].breakdowns[]` | Child breakdowns | Nested up to four levels deep, ending in `Base` and tax lines. |
| `contexts[].contextType` | Which kind of context this is | Values include: `ProductContext` (items only), `AddressContext`, `DeferredContext`, `PaymentsContext`, `TimeRangeContext` (items only). |
| `contexts[].asin` | Product context: the item's ASIN | |
| `contexts[].sku` | Product context: the item's SKU | Special characters come through as plain text (a `£` is `£`). A `"` inside a SKU has a backslash before it (`\"`), even after JSON parsing: remove it before joining on SKU. |
| `contexts[].quantityShipped` | Product context: quantity shipped | Integer; `null` on some `ServiceFee` items. |
| `contexts[].fulfillmentNetwork` | Product context: fulfilment network | e.g. `AFN`. |
| `contexts[].deferralReason` | Deferred context: deferral policy | Amazon's examples: `DD7` (delivery date policy) and `B2B` (invoiced orders). |
| `contexts[].maturityDate` | Deferred context: when the transaction is released | ISO 8601 date-time, UTC. |
| `contexts[].startTime` | Time range context: start of the transaction's time range | ISO 8601. Rare: only on a few item contexts. |
| `contexts[].endTime` | Time range context: end of the transaction's time range | ISO 8601. |

Documented but not seen in real files: the Amazon Pay and Business context fields (`storeName`, `orderType`, `channel`), and the payments context fields (`paymentType`, `paymentMethod`, `paymentReference`, `paymentDate`). `PaymentsContext` and `AddressContext` appear with `contextType` only, no other fields.

## Downloaded from Seller Central

**A different file from the API's.** `listTransactions` sends JSON with every breakdown. Transaction View's **Download** button gives a short CSV of the rows on screen. Amazon's banner says the download is limited to 600 transactions, and points to All Statements or Reports Repository for the complete list and the columns not shown. The screen itself stops loading at 10,000 transactions. For a whole period, use the API, or the Reports Repository's date range report ([`date_range_transactions`](date_range_transactions.md)).

Seen in a real UK seller's Seller Central download (Oct 2026), for one week:

- **File name:** `Transactions for time period DD_MM_YYYY - DD_MM_YYYY.csv`, named by the period.
- **Format:** UTF-8 with a byte order mark; every field in double quotes.
- **10 columns:** `Date`, `Transaction status`, `Transaction type`, `Order ID`, `Product Details`, `Total product charges`, `Total promotional rebates`, `Amazon fees`, `Other`, `Total (GBP)`.
- **Dates** are DD/MM/YYYY.
- **Amounts** are plain numbers with no `£`, written with as few decimals as needed (`0`, `-1.5`, `12.34`), not a fixed two. The currency is only in the `Total (GBP)` header.
- **`Product Details` is cut short** with `...`, so it won't match a full title.
- No buyer details.

Against the API, it has no transaction id, settlement id, SKU, ASIN or store column, and only four amount columns before the total. So it can be matched to other reports only by `Order ID`, and can't be split by fee type. For traps shared by all Seller Central downloads, see [seller-central-downloads](seller-central-downloads.md).

## Personal data

None seen: `AddressContext` and `PaymentsContext` came with no fields. If Amazon ever fills the address or payment context fields, strip them before analysis and never copy them into anything shared. Some sellers put their cost price in the SKU, so treat SKUs as private.

## Questions it answers

- What did each order actually earn after every fee, item by item (`items[].breakdowns`)? Count each transaction once (see the deferred pairs below).
- How much money is deferred right now, and when does it release (`transactionStatus`, `contexts[].maturityDate`)?
- Which transactions make up a given payout? Amazon's tutorial takes the `RELEASED` transactions with the payout's `FINANCIAL_EVENT_GROUP_ID` ([Determine payment transactions](https://developer-docs.amazon/sp-api/docs/determine-payment-transactions)); never sum every line, because each released deferred transaction has two (see below). Leave out the transactions whose `transactionType` is `Transfer` too: they are `RELEASED`, positive, with `description` `Disbursement`, and they carry an earlier payout, usually the same store's group just before. Filter on `transactionType`, not `description`: some `Adjustment` transactions are also `Disbursement`, and they belong to their payout. Without the `Transfer` transactions, a settlement's `RELEASED` transactions, grouped by `SETTLEMENT_ID` (one to one with `FINANCIAL_EVENT_GROUP_ID`), added up exactly to its `total-amount` in [`settlements`](settlements.md) in real data, once the settlement file's `Payable to Amazon` row (a balance carried in), where it has one, is added. With the `Transfer` transactions left in, the totals didn't match.
- What were the month's refunds, and what did each one cost?
- Which non-order charges (storage, removals, inbound fees) hit the account in a month? Total them one `currencyCode` at a time: a UK seller with stock in EU centres is charged storage in EUR as well as GBP (see the storage trap below).

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Order transactions are often deferred** before release, under Amazon's delivery-date policy (`DD7`) or as invoiced orders (`B2B`).
- **A released deferred transaction has two lines.** The original turns `DEFERRED_RELEASED`, and the release is a separate `RELEASED` transaction with the same amount, posted later, often under a later settlement. Summing every line counts those twice. Usually the original names its release in `RELEASE_TRANSACTION_ID` (for example on `Shipment` and `Refund` transactions). But older pairs can name nothing, though a `RELEASED` twin with the same `ORDER_ID`, type and amount exists. So rules that rely on `RELEASE_TRANSACTION_ID` double-count those pairs.
- **For order earnings, count each transaction once, and leave out every `Transfer` transaction** (they carry earlier payouts, not earnings; see the payout question above): count the `RELEASED` lines, leave out every `DEFERRED_RELEASED` line, and count a `DEFERRED` line only if no `RELEASED` line releases it. A release names its original in `DEFERRED_TRANSACTION_ID`. Match the older unlinked ones one to one. Where both lines carry a `SHIPMENT_ID` (shipments) or a `REFUND_ID` (refunds), match on that. Otherwise, within one `ORDER_ID`, `transactionType` and `totalAmount`, each `RELEASED` line releases only one deferred line, and any extra deferred lines with the same type and amount are still deferred. The money total is then right, but which of those lines was released can't be told, so mark their per-shipment or per-refund figures as unresolved: never match two deferred lines to the same release. This matters when you keep copies by month: a `DEFERRED` line in an older copy can be stale while its release sits in a later month. A deferred amount is dated by its release, which often falls in a later month than the original. For payouts, use the `RELEASED` lines other than `Transfer` (see the payout question above).
- **Storage fees moved in the breakdowns in May 2026.** On `FBAStorageBilling` items the fee before tax is the `Base` under `AmazonFees > FBAStorageFee` up to April 2026, and under `FBAFees > StorageBillingFee > FBAStorageFee` from May 2026. Long-term storage (`FBALongTermStorageBilling`) moved the same way, from `AmazonFees > FBALongTermStorageFee` to `FBAFees > LongTermStorageFee > FBALongTermStorageFee`. Sum every `Base` under a storage item, whatever its path: no item has both paths, and `Base` plus `Tax` equals the item's total. **Sum one currency at a time:** a UK seller with stock in EU centres gets EUR storage lines beside the GBP ones, and summed as one they distort the month's storage figure.
- **Amazon keeps the VAT on some exports.** Some `Shipment` transactions have a `MarketplaceFacilitatorVAT-Principal` line under `Tax`, in the UK store or in EU stores. In the UK store these can be orders sent abroad: check `ship-country` in [`orders`](orders.md), and note that an order to the Channel Islands can show `ship-country` `GB`, with only the postcode to tell. The line equals minus `OurPriceTax` on nearly every such item, and the item's `Tax` total nets to 0; where a transaction has several items, their `Tax` totals net to 0 together. So summing `OurPriceTax` overstates the VAT the seller collected (and, on standard VAT accounting, owes), and `ProductCharges` plus `OurPriceTax` overstates what the seller received. For VAT, use the item's `Tax` total: on every item it is the sum of the lines under it, facilitator lines included. The same goes for `MarketplaceFacilitatorVAT-Shipping`, any `MarketplaceFacilitatorTax-…` lines, and refunds (a `Refund` with a `MarketplaceFacilitatorVAT-Principal` line nets its `Tax` to 0).
- **No addresses or payment details:** `AddressContext` and `PaymentsContext` come with no fields.
- Every transaction has `SETTLEMENT_ID` and `FINANCIAL_EVENT_GROUP_ID`, so transactions can be grouped by settlement even for weeks with no settlement report.

Seen in Seller Central downloads (Oct 2026):

- **Transaction View's download is capped.** It holds at most 600 transactions, as Amazon's banner says, and the screen stops loading at 10,000. A busy period comes back cut short, so take period totals from the API or the date range report.

Sources: [Finances API v2024-06-19 Reference](https://developer-docs.amazon/sp-api/docs/finances-api-v2024-06-19-reference), the model [finances_2024-06-19.json](https://github.com/amzn/selling-partner-api-models/blob/main/models/finances-api-model/finances_2024-06-19.json), [Get the latest transactions](https://developer-docs.amazon/sp-api/docs/get-latest-transactions) for `breakdownType`, [listTransactions](https://developer-docs.amazon/sp-api/reference/listtransactions), [Determine payment transactions](https://developer-docs.amazon/sp-api/docs/determine-payment-transactions).
