# SKU economics (Data Kiosk Seller Economics) (`seller_economics`)

Amazon's per-SKU economics for a period: sales, every fee by type, ad spend, the costs the seller entered, and net proceeds, for every store.

## Where to find it in Seller Central

Checked in Seller Central UK on 28 Oct 2026: **Finance > Profit analytics > Selling economics**, then the **SKU Economics report** tab. In classic Seller Central it is **Reports > SKU Economics report** (or Reports > Selling economics and fees). The report covers sales, fees, ads, per-unit costs and net proceeds by parent ASIN, child ASIN, MSKU or FNSKU ([SKU economics report](https://sellercentral.amazon.co.uk/help/hub/reference/external/GZ8Y22NL2FSRY8M5)). Amazon's schema links that page for the fee types it supports.

To request it: first choose the store (several can be ticked), the aggregation level (MSKU by default) and the period (last 7 or 30 days, a month, a quarter, a year, or a custom range). Then tick what to include: sales, fees, advertising spend, off-Amazon costs, storage fee base rate and surcharges, fulfilment base rate and surcharges, and net proceeds. Then **Generate report**. Amazon says the download links stay active for 30 days only, and shows when each report was made in your browser's time zone. The same page has a **Fees and Economics Preview Report** tab and **Additional financial inputs**, where you can enter your own costs.

**Profit analytics** sits in the same top bar as Selling economics, beside Scenario Modelling and the Revenue Calculator. Amazon's banner calls it a new reporting tool, and it shows similar figures on screen ([Profit analytics](https://sellercentral.amazon.co.uk/help/hub/reference/external/G8CXLSH94WNPB9R4)).

## Getting it through the API

- **Call / report type:** Data Kiosk API 2023-11-15, dataset `analytics_economics_2024_03_15`: "aggregated sales, fees, and cost for selected period", refreshed daily ([schema](https://github.com/amzn/selling-partner-api-models/blob/main/schemas/data-kiosk/analytics_economics_2024_03_15.graphql)). The September 2026 release added deal, coupon, removal, disposal and fuel fees ([release notes](https://developer-docs.amazon/sp-api/docs/sp-api-release-notes)).
  1. Check with `getQueries` that no query is `IN_QUEUE` or `IN_PROGRESS`: Amazon refuses a new query while the same one is pending ([Data Kiosk API](https://developer-docs.amazon/sp-api/docs/data-kiosk-api)).
  2. `createQuery` with a GraphQL query. Amazon's limit is 8000 characters after whitespace is removed ([createQuery](https://developer-docs.amazon/sp-api/reference/createquery)).
  3. Poll `getQuery` until it is done, then `getDocument` for the data document (JSON Lines, unzip it if gzipped). `DONE` with no document means no data for the range ([Data Kiosk API](https://developer-docs.amazon/sp-api/docs/data-kiosk-api)).
  4. If the query comes back with `pagination.nextToken`, send `createQuery` again with that token and the query text Amazon returned, and join the documents.
- **Query arguments:** `startDate` and `endDate`, `marketplaceIds` (the stores you want), `aggregateBy: { date: RANGE, productId: MSKU }`, and `includeComponentsForFeeTypes: [FBA_FULFILLMENT_FEE, FBA_STORAGE_FEE]` (the only two fee types the schema allows there). `RANGE` sums over the whole requested range, so ask one calendar month at a time for monthly figures ([schema](https://github.com/amzn/selling-partner-api-models/blob/main/schemas/data-kiosk/analytics_economics_2024_03_15.graphql)).
- **Role:** [Brand Analytics](https://developer-docs.amazon/sp-api/docs/data-kiosk-api) for `createQuery`, `getQuery`, `getQueries` and `getDocument`. Amazon also checks a role for each field at query creation, but names no per-field roles for this dataset ([Data Kiosk API](https://developer-docs.amazon/sp-api/docs/data-kiosk-api)).
- **How far back Amazon goes:** the start date "must be no more than two years ago" ([schema](https://github.com/amzn/selling-partner-api-models/blob/main/schemas/data-kiosk/analytics_economics_2024_03_15.graphql)). Every field carries `@resultRetention(duration: "P30D")`, so Amazon keeps each query and its document for 30 days ([createQuery](https://developer-docs.amazon/sp-api/reference/createquery)); save your own copy.

## Columns

JSON Lines: one line is one record, one SKU in one store for the period. Amazon's docs don't show what one line looks like; each record carries its own `marketplaceId`, `msku`, `startDate` and `endDate`.

Money fields are objects with `amount` (a number) and `currencyCode` (ISO 4217, the store's currency). An **aggregated detail** is an object with `amount`, `amountPerUnit`, `amountPerUnitDelta`, `promotionAmount`, `quantity`, `taxAmount` and `totalAmount`, described once under `fees[].charges[].aggregatedDetail`. Lists are marked `[]`.

| Column | Meaning | Format and traps |
|---|---|---|
| `startDate` | Start of the period the record covers | `YYYY-MM-DD`. |
| `endDate` | End of the period the record covers | `YYYY-MM-DD`. |
| `marketplaceId` | Store the record is for | Amazon's store id. |
| `msku` | The seller's SKU | |
| `fnsku` | Amazon's fulfilment SKU | **`null` on every record** in real files; join on `msku` instead. |
| `childAsin` | Child ASIN (the sellable product) | |
| `parentAsin` | Parent ASIN | |
| `sales.orderedProductSales` | Price times units ordered in the period | Money. **Its VAT basis changes:** VAT included up to about 24 Aug 2025, left out from about 25 Aug 2025. **Doubled on some SKU-months** from that change to January 2026. See the traps for both. |
| `sales.unitsOrdered` | Units ordered | Can be `0`: SKUs with fees but no sales. **Double the real count on some SKU-months** up to January 2026 (see the traps). |
| `sales.averageSellingPrice` | `orderedProductSales` divided by `unitsOrdered` | Money; **`0.0`, not null,** when `unitsOrdered` is 0. Half the real price where `unitsOrdered` is doubled but sales aren't (up to August 2025). |
| `sales.refundedProductSales` | Refunded product sales | Money; counted by refund request time. **Leaves out VAT in every month,** also while `orderedProductSales` includes it. **Doubled on some SKU-months** up to January 2026. See the traps for both. |
| `sales.unitsRefunded` | Units refunded in the period | **Double the real count on some SKU-months** up to January 2026 (see the traps). |
| `sales.netProductSales` | `orderedProductSales` minus `refundedProductSales` | Money. **Mixes VAT bases up to about 24 Aug 2025:** sales with VAT minus refunds without it. From about 25 Aug 2025 both leave VAT out. |
| `sales.netUnitsSold` | `unitsOrdered` minus `unitsRefunded` | Can be negative. |
| `fees[].feeTypeName` | Fee type | Code names, not the "FBA fulfillment fees" wording of Amazon's schema example, such as `ReferralFee`, `RefundedReferralFee`, `FbaFulfilmentFee`, `FbaStorageFee` and `FBAInventoryReimbursement`. Amazon's schema points to the [SKU economics report](https://sellercentral.amazon.co.uk/help/hub/reference/external/GZ8Y22NL2FSRY8M5) help page for the fee types. `FBAInventoryReimbursement` is a credit, already counted in `netProceeds`: adding the [`reimbursements`](reimbursements.md) report on top counts it twice. |
| `fees[].charges[].identifier` | Unique id for this fee | The fee type plus `_Standard` (e.g. `FbaStorageFee_Standard`). |
| `fees[].charges[].startDate` | Start of the fee's date range | `YYYY-MM-DD`. Seen never split: one charge per fee type in every record. |
| `fees[].charges[].endDate` | End of the fee's date range | `YYYY-MM-DD`. |
| `fees[].charges[].aggregatedDetail.amount` | Amount from the rate card | Money. Before VAT. |
| `fees[].charges[].aggregatedDetail.promotionAmount` | Promotion amount, taken off `amount` | Money. |
| `fees[].charges[].aggregatedDetail.taxAmount` | Tax amount, added to `amount` | Money. |
| `fees[].charges[].aggregatedDetail.totalAmount` | Final charge: `amount` - `promotionAmount` + `taxAmount` | Money. Positive for a charge; negative for a credit (as an `FBAInventoryReimbursement` charge usually is). |
| `fees[].charges[].aggregatedDetail.quantity` | Charged quantity | A float (`1.0`); never null in real files. |
| `fees[].charges[].aggregatedDetail.amountPerUnit` | `totalAmount` divided by `quantity` | Money; occasionally `null`. Includes VAT. |
| `fees[].charges[].aggregatedDetail.amountPerUnitDelta` | Change in `amountPerUnit` across an Amazon fee change | `null` on every charge in real files. |
| `fees[].charges[].properties[]` | Inputs to the fee (`propertyName`, `propertyValue`) | Always an empty list in real files. |
| `fees[].charges[].components[].name` | Fee component | Only for the two fee types asked for: `BaseMonthlyStorageFee` and `UtilizationSurchargeFee` (storage), `BaseFbaFulfilmentFee` and `FuelSurcharge` (fulfilment). `null` for other fees. |
| `fees[].charges[].components[].aggregatedDetail` | The component's figures | Aggregated detail. |
| `fees[].charges[].components[].properties[]` | Inputs to the component | Always empty in real files. |
| `ads[].adTypeName` | Ad type | Amazon's schema example is "Sponsored Products charge". `ads` is `[]` for a SKU with no ad spend. |
| `ads[].charge` | Ad spend for that type | Aggregated detail. Carries no VAT, though Amazon's advertising invoice can add VAT on top (see the `netProceeds` trap). |
| `cost.costOfGoodsSold`, `cost.fbaCost`, `cost.mfnCost`, `cost.miscellaneousCost` | Seller-entered costs per unit | What the seller entered in SKU Central or the Revenue Calculator; null when nothing was entered ([schema](https://github.com/amzn/selling-partner-api-models/blob/main/schemas/data-kiosk/analytics_economics_2024_03_15.graphql)). **`null` on every record when no costs are entered**, and then `fbaCost` and `mfnCost` have no sub-fields. |
| `netProceeds.total` | `netProductSales` - all fee `totalAmount`s - all ad `totalAmount`s - per-unit cost x `netUnitsSold` | Money. Matched the formula on every record checked, so fee credits are already in. Negative for a SKU with only fees. With no costs entered, it's before the cost of goods. **Mixes VAT bases:** fees come off with their VAT, while the sales leave VAT out from about 25 Aug 2025 and the refunds in every month (see the traps). Wrong on doubled SKU-months: too low up to the VAT change, where only the refunds are doubled in money, and too high from it to January 2026, where the sales are doubled too. |
| `netProceeds.perUnit` | `netProceeds.total` divided by `netUnitsSold` | Money. **Don't use it.** With no net units it's mostly `0.0`, only sometimes `null`. On most records with negative `netUnitsSold` it's positive while the total is negative, so losers look like earners. Divide `total` by `netUnitsSold` yourself, only when that's above 0, and not on a doubled SKU-month. |

Every field the query asks for is in the file, except `cost`'s sub-fields (`cost.fbaCost.shippingToAmazonCost`, `cost.mfnCost.fulfillmentCost`, `cost.mfnCost.storageCost`), which don't appear when their parents are `null`. `fees[].charge` is left out because Amazon deprecated it in favour of `charges`.

## Downloaded from Seller Central

**A different file from the API's.** The Data Kiosk dataset sends JSON Lines with nested fields. Seller Central's SKU Economics report is a flat CSV with plain-English column names, so no field name matches. Seen in a real UK seller's Seller Central download (Oct 2026), for one store, by MSKU, for one calendar month, with every box ticked:

- **File name:** a long generated id ending `.amzn1.tortuga.4.eu.csv`, not the period.
- **Format:** comma-separated, UTF-8 with a byte order mark, lines ending in LF. 69 columns.
- **Dates** (`Start date`, `End date`) are **MM/DD/YYYY**, US order, not the API's `YYYY-MM-DD`.
- **Numbers** are plain, with no currency sign; fees are positive numbers. `Currency code` names the currency.
- **`Amazon store`** is a country code (`GB`), not the API's marketplace id.
- **One row per MSKU in the catalogue**, including those with no sales in the month, as in the API's records with no activity.
- **`FNSKU` is blank** at MSKU level, as `fnsku` is null in the API.
- **Costs are blank** unless you have entered them (Additional financial inputs, or the off-Amazon costs box).

Columns, in order: `Amazon store`, `Start date`, `End date`, `Parent ASIN`, `ASIN`, `FNSKU`, `MSKU`, `Currency code`, `Average sales price`, `Units sold`, `Units returned`, `Net units sold`, `Sales`, `Net sales`. Then three columns for each fee, `<fee> per unit`, `<fee> quantity` and `<fee> total`, for these fees: Aged inventory surcharge, Base fulfilment fee, Base monthly storage fee, Digital Services Fee (FBA Fulfilment fees), Digital Services Fee (Selling on Amazon fees), FBA Inventory Reimbursement, FBA disposal order fee, FBA removal order fee, Fuel and logistics-related surcharge, Fulfilment by Amazon fulfilment fees, Inbound Transportation Fee, Inbound Transportation Program Fee, Monthly storage fee, Referral Fee Refunds, Referral fee, Refund administration fee, Storage utilisation surcharge. Then `Cost of goods sold per unit`, `Miscellaneous cost per unit`, `Net proceeds total` and `Net proceeds per net unit sold`.

Check before using it:

- **The advertising column can be missing,** even with that box ticked. Don't count on one.
- **Fee parts may sit beside their totals.** By name, `Base fulfilment fee` and `Fuel and logistics-related surcharge` look like the parts of `Fulfilment by Amazon fulfilment fees`, and `Base monthly storage fee` and `Storage utilisation surcharge` like the parts of `Monthly storage fee`, as the API's `components` are. This wasn't checked. Test whether the parts add up to the total before summing every fee column, or those fees may be counted twice.
- **The API file's traps weren't checked here** (the VAT basis, the doubled SKU-months, `perUnit`). Check the CSV the same way before trusting it.

For traps shared by all Seller Central downloads, see [seller-central-downloads](seller-central-downloads.md).

## Personal data

None of the columns hold buyer or business personal details. Some sellers put their cost price in the SKU, so treat SKUs as private.

## Questions it answers

- What did each SKU make after Amazon's fees in each store last month, and per unit (divided by `netUnitsSold`, when that's above 0)? `netProceeds` takes the fees off with their VAT, and what to do with that depends on the seller's VAT position (see the VAT section of [SKILL.md](../SKILL.md)). A seller who reclaims fee VAT (VAT-registered on standard accounting) wants a figure before VAT: take `netProceeds.total` and add back every fee's `taxAmount`. A Flat Rate Scheme or non-registered seller can't reclaim it, so their cost is the fees with VAT: don't add `taxAmount` back. A Flat Rate seller's profit doesn't come from this dataset, as its sales leave out the VAT they collected: see the `netProceeds` trap. Up to about 24 Aug 2025 `orderedProductSales` holds VAT as well, and up to January 2026 some SKU-months have doubled units and refunds: check both first (see the traps).
- Which SKUs paid the most in FBA fulfilment and storage fees, and how did those split into base fee and surcharges (`components`)?
- Did an Amazon fee change raise a SKU's per-unit fee? `amountPerUnitDelta` is `null` on every charge, so it can't answer this. Comparing `amountPerUnit` across monthly records doesn't isolate a fee change either: the referral fee's moves with the selling price, and the storage fee's moves with the season.
- Which SKUs have a high refund count against units ordered? Up to January 2026, leave out doubled SKU-months first (see the traps): `unitsRefunded` is doubled along with `unitsOrdered`, and also in some months with no sales, which inflates a rate taken over several months.
- If cost of goods is entered in SKU Central, which SKUs lose money after all fees?
- What referral fee was actually charged? The `ReferralFee` charge (`RefundedReferralFee` gives refunds back). [`fba_fee_preview`](fba_fee_preview.md) has only the estimate, which leaves VAT out: compare it with `amount` divided by `quantity`, not with `amountPerUnit` or `totalAmount`, which include the VAT. What was actually charged before VAT is `amount` minus `promotionAmount`.

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Records with no activity.** A month can list every store and SKU on the account, including records with no units, sales, fees, ads or proceeds. Amazon documents no row limit, so a cap isn't ruled out. Count records with sales, fees or ads, not records.
- **Matching SKUs with other reports.** Every SKU in [`reimbursements`](reimbursements.md), [`fba_returns`](fba_returns.md) and [`orders`](orders.md) was here, once two things were done: decode the HTML entities in the other reports' SKUs (`&pound;`, `&amp;`, `&quot;`), and leave out cancelled lines and any Multi-Channel Fulfilment lines in `orders` (a SKU found only on those lines can be missing here). [`transactions`](transactions.md) agreed once its `\"` was read as `"`; a SKU it has that isn't here can be on items other than sales and refunds, such as `ServiceFee` items.
- **Refunds go missing once a SKU drops out of the list.** Some SKU-months have a `Refund` in `transactions` but no record here, so those refunds are in no month of this dataset. That happens when the refund comes after the SKU's last record here, in a month with no sales of it, for an order placed in an earlier month. For a SKU-month with a `Refund` in `transactions` but no record here, take the refund from `transactions`.
- **No costs unless entered:** with no costs in SKU Central, `cost`'s four sub-fields are null on every record, so `netProceeds` is after Amazon's fees but before the cost of goods.
- **Some SKU-months are counted twice.** On some ASIN-months up to January 2026, `unitsOrdered` was exactly double Business Reports ([`sales_and_traffic`](sales_and_traffic.md), per child ASIN per month); none from February 2026. It's per SKU: each such SKU's `unitsOrdered` is double its own quantity in `orders`, also where an ASIN has several SKUs. No record is repeated, so it's in Amazon's figures. Up to the VAT change in late August 2025 the sales on those records are right, so `averageSellingPrice` is half the price; from the change, sales are doubled too. **Refunds are doubled as well, before and after the change:** on some SKU-months up to January 2026 (none from February 2026), `unitsRefunded` is twice the refunded units in `transactions`, and `refundedProductSales` doubles with it, including SKU-months with refunds but no sales. Fees are counted once (their `quantity` matches `orders`, and the refund fees' `quantity` matches `transactions`). So per-unit profit, average price and refund rates are wrong for those SKUs. Up to the change the extra refunds make `netProductSales` and `netProceeds` too low. From September 2025 to January 2026 monthly totals are too high: sales are doubled on those SKUs, and `netProceeds` by a larger share, as the fees aren't doubled. Until January 2026, check each SKU-month before using it, in two ways:
  - Check `unitsRefunded` against the quantity of the SKU's `Refund` items in `transactions` that month, counting each refund once and dating it by its first line (a deferred refund's `DEFERRED_RELEASED` line, not its later release; see `transactions`).
  - Check `unitsOrdered` against `orders`. For a month your `orders` data covers only in part, check against `sales_and_traffic` per child ASIN instead: add up a child ASIN's entries first, as it can be listed under more than one parent, and note that Business Reports counts days in UK time.
- **Sales drop VAT in late August 2025.** Per SKU against `orders`, UK `orderedProductSales` matched the VAT-inclusive `item-price` before about 25 Aug 2025 and `item-price` less `item-tax` after it. A zero-rated item (`item-tax` 0) carries no VAT, so it looks unchanged. Up to July 2025, UK monthly `orderedProductSales` was close to Business Reports' VAT-inclusive sales.
- **Fees carry their VAT throughout,** in each charge's `taxAmount`, though not every charge has VAT: ads carry none, nor do `FBAInventoryReimbursement` charges, and some `FbaStorageFee` charges, real fees among them, have `taxAmount` 0. On UK `ReferralFee` charges `taxAmount` was 20% of `amount`.
- **Refunds leave VAT out in every month.** Leaving out the doubled SKU-months, UK `refundedProductSales` matched `transactions`' before-VAT `OurPricePrincipal`, and never with `OurPriceTax` added. So up to the change `netProductSales` is sales with VAT minus refunds without it, and across the change it drops by the VAT on `orderedProductSales` (1/6 of a VAT-inclusive price on 20% items).
- **`netProceeds` mixes VAT bases.** From the change it is sales and refunds without VAT minus fees with VAT. Up to the change it also holds the VAT on `orderedProductSales`, and its false drop across the change is that VAT. What to do depends on the seller's VAT position (see the VAT section of [SKILL.md](../SKILL.md)):
  - **A seller who reclaims fee VAT** (VAT-registered on standard accounting): `netProceeds` understates their result before VAT by the fee VAT, and up to the change overstates it by the VAT on the sales. Add every fee's `taxAmount` back to `netProceeds.total`, and up to July 2025 also take off the VAT inside `orderedProductSales`. Taking off the VAT on `netProductSales` instead takes off too little, as the refunds never held VAT.
  - **A seller who isn't VAT-registered** can't reclaim fee VAT, so the fees with VAT are their real cost: don't add `taxAmount` back. Ad charges here leave out any VAT that Amazon adds on its advertising invoice, which it can charge when the seller hasn't given it a VAT number ([Amazon Ads: digital ad taxes](https://advertising.amazon.com/en-gb/library/guides/digital-ad-taxes)): take that VAT from the ads invoices and count it as a cost too.
  - **A Flat Rate Scheme seller** can't reclaim fee VAT either, so don't add `taxAmount` back. The VAT they owe isn't in this dataset: it's a flat percentage of their turnover, worked out on their VAT return (see the VAT section of [SKILL.md](../SKILL.md)). Take it from the seller or their accountant rather than rebuilding VAT-inclusive turnover from these records, which mix VAT bases. `netProceeds` doesn't give their profit either: their sales count with the VAT they collected, and from the change this dataset leaves that VAT out. Say so, and work from [`transactions`](transactions.md) instead: each transaction's total is what Amazon paid, sales with their VAT minus fees with theirs (count each transaction once, as that page says), then take off the VAT they owe as their return or accountant gives it. Leave out `Transfer` transactions, which carry earlier payouts, and any payment the seller made to clear a negative balance (a charge to their card or bank account, which settlement files show as a positive `Successful charge` row): it is their own money paying Amazon, not earnings, and the fees it paid for stay in. Amazon doesn't list every `transactionType`, so check any positive transaction that isn't an order, a refund or a reimbursement before counting it. Those totals are before the cost of goods and any other costs entered in SKU Central, so take those off too, or label the result as before them. Advertising: ads paid from the Amazon balance are already in those totals, with any VAT Amazon charged on them, so take nothing more off for them. Ads paid by card or invoice outside the account aren't in the totals: take those invoices off as well, with any VAT on them ([Amazon Ads: digital ad taxes](https://advertising.amazon.com/en-gb/library/guides/digital-ad-taxes)), or label the result as before advertising.

  **August 2025 holds both bases.** That month's sales hold VAT up to about 24 Aug only. Add up `orders`' `item-tax` on that SKU's orders with a `purchase-date` up to then to take it off for a seller who reclaims fee VAT. As the exact day of the change isn't known, the result is close, not exact. Or mark August 2025 as mixed and leave it out of comparisons. For long comparisons, use `transactions`, whose `ProductCharges` are net of VAT where Amazon calculates the VAT (see that page).
- **`fnsku` is always null.** Join on `msku`.

Seen in Seller Central downloads (Oct 2026):

- **The SKU Economics CSV uses US dates** (MM/DD/YYYY) in a UK account, and lists base fees and surcharges beside the fee totals they may belong to (see Downloaded from Seller Central).

Sources: [analytics_economics_2024_03_15.graphql](https://github.com/amzn/selling-partner-api-models/blob/main/schemas/data-kiosk/analytics_economics_2024_03_15.graphql), [Data Kiosk API](https://developer-docs.amazon/sp-api/docs/data-kiosk-api), [createQuery](https://developer-docs.amazon/sp-api/reference/createquery), [SP-API release notes](https://developer-docs.amazon/sp-api/docs/sp-api-release-notes). The UI version of the same terms is on [SKU economics report](https://sellercentral.amazon.co.uk/help/hub/reference/external/GZ8Y22NL2FSRY8M5).
