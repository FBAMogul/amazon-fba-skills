# Fee Preview (`fba_fee_preview`)

Amazon's estimated referral and FBA fees per SKU for your FBA stock with active offers, which Amazon updates at least once every 72 hours ([SP-API FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)); the UK help says the data "may be up to 72 hours old" ([Fee Preview Report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201115050)). Fees are estimates at the current price and dimensions, not charges. **The price includes VAT, the fees leave it out:** Amazon adds VAT (20% in the UK) to its fees when it charges them (see Columns).

## Where to find it in Seller Central

**Supply chain > Supply chain reports** (the Amazon Fulfilment Reports page; [Amazon Fulfilment reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201104750)), then **Payments > Fee Preview** in the left-hand list (`/reportcentral/ESTIMATED_FBA_FEES/1`, the address the UK help links: [Fee Preview Report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201115050)). Classic: **Reports > Fulfilment by Amazon**. Checked in Seller Central UK on 28 Oct 2026.

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_FBA_ESTIMATED_FBA_FEES_TXT_DATA`, every store's marketplace id, and `dataStartTime` at least 72 hours before now (for example midnight UTC three days ago) with no `dataEndTime`, and no report options; then poll `getReport` until it's done and fetch the file with `getReportDocument`. Amazon wants `dataStartTime` at least 72 hours before now and `dataEndTime` at now ([SP-API FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)); a missing `dataEndTime` defaults to now ([Reports API model](https://github.com/amzn/selling-partner-api-models/blob/main/models/reports-api-model/reports_2021-06-30.json)).
- **Role:** Amazon Fulfillment or Pricing ([role mappings](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)).
- **Limit:** once per day per seller ([SP-API FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba)). A second request the same day comes back CANCELLED ([Reports FAQ](https://developer-docs.amazon/sp-api/docs/reports-faq)); that isn't a failure, so keep the day's first copy.
- **How far back Amazon goes:** not stated in Amazon's docs. Each file is the estimate as it stood when asked for, so there is no back history: it starts with the first copy you save.

## Columns

Checked against a real UK seller's file in Oct 2026: 32 columns, in this order. Tab-separated text, header row first. Amazon writes `--` for "not applicable".

| Column | Meaning | Format and traps |
|---|---|---|
| `sku` | Your SKU. | HTML entities: `£` comes as `&pound;`, `&` as `&amp;`, `–` as `&ndash;`. Decode them before joining to `inventory_all`, which has plain characters. |
| `fnsku` | Amazon's identifier for the unit stored in and fulfilled from its fulfilment centres. | |
| `asin` | Amazon's product identifier. | |
| `amazon-store` | Not defined in Amazon's docs; the store the estimate is for. | Two-letter code, e.g. `GB`. |
| `product-name` | Product title. | |
| `product-group` | Amazon department or category. | Plain names, not the `gl_…` codes in `inventory_planning`. |
| `brand` | Not defined in Amazon's docs; the product's brand. | |
| `fulfilled-by` | Who fulfils the item. | `Amazon`, not MFN/AFN. |
| `has-local-inventory` | Not defined in Amazon's docs. | `true` or `false`. |
| `your-price` | Your current selling price. | **Includes VAT.** Checked against `orders`: where a UK order line matched it, it matched the VAT-inclusive unit price, never the net price. |
| `sales-price` | Your current promotional price. | Equals `your-price` when no sale runs, so includes VAT too. |
| `longest-side` | Longest side including packaging. | Decimal, in `unit-of-dimension`. |
| `median-side` | Side that is neither longest nor shortest, including packaging. | Decimal. |
| `shortest-side` | Shortest side including packaging. | Decimal. |
| `length-and-girth` | Longest side plus girth, girth = 2 x (median + shortest side). | Decimal. |
| `unit-of-dimension` | Unit for the dimensions. | `centimeters`. |
| `item-package-weight` | Weight including packaging. | Decimal, in `unit-of-weight`. |
| `unit-of-weight` | Unit for the weight. | `grams` here, kilograms in `storage_fees`. |
| `product-size-weight-band` | Not defined in Amazon's docs; the FBA size and weight band. | Band names with no spaces, such as `SmallParcel`. |
| `currency` | Currency the fee is calculated in. | e.g. `GBP`. |
| `estimated-fee-total` | Estimated FBA plus selling fee per unit. | In `currency`. Referral fee + domestic fulfilment fee + variable closing fee, without the Digital Services Fee. **Before VAT,** like every fee column here, while `your-price` includes it. |
| `estimated-referral-fee-per-unit` | Estimated referral fee. | The best per-SKU referral fee estimate for a stocked SKU. Worked out on the VAT-inclusive price (a category rate of `your-price`, often 15%), but before VAT on the fee: it matched `transactions`' `Commission` > `Base` on single-unit UK sales at `your-price`, never `Base` plus its `Tax`. |
| `estimated-variable-closing-fee` | Estimated variable closing fee. | `0.00` when none applies. |
| `estimated-fixed-closing-fee` | Not in Amazon's EU column list. | Can be `--` on every row. |
| `estimated-order-handling-fee-per-order` | Estimated order handling fee per order. | Can be `--` on every row. |
| `expected-domestic-fulfilment-fee-per-unit` | Estimated fulfilment fee from the item's dimensions and weight. | Before VAT: it usually matched `transactions`' `FBAPerUnitFulfillmentFee` > `Base` on single-unit UK sales, and never `Base` plus its `Tax`. Some sales were charged more than the estimate. |
| `expected-efn-fulfilment-fee-per-unit-uk` | Not defined in Amazon's docs (they give the name only); by name, a European Fulfilment Network fee involving the UK. | Can be `--` on every row. |
| `expected-efn-fulfilment-fee-per-unit-de` | Not defined in Amazon's docs; by name, the same for Germany. | Seen equal to `expected-domestic-fulfilment-fee-per-unit` on every row. Don't read it as a cross-border cost. |
| `expected-efn-fulfilment-fee-per-unit-fr` | Not defined in Amazon's docs; by name, the same for France. | Seen equal to the domestic fee on every row, like `-de`. |
| `expected-efn-fulfilment-fee-per-unit-it` | Not defined in Amazon's docs; by name, the same for Italy. | Seen equal to the domestic fee on every row, like `-de`. |
| `expected-efn-fulfilment-fee-per-unit-es` | Not defined in Amazon's docs; by name, the same for Spain. | Seen equal to the domestic fee on every row, like `-de`. |
| `expected-efn-fulfilment-fee-per-unit-se` | Not defined in Amazon's docs; by name, the same for Sweden. | Seen equal to the domestic fee on every row, like `-de`. |

Documented but not in the file: `product-size-tier`.

## Personal data

None: no buyer or business details. Some sellers put their cost price in the SKU, so treat SKUs as private. Rule: strip personal data before analysis and never copy it into anything shared.

## Questions it answers

- What will Amazon take per unit (referral, fulfilment and any variable closing fee: `estimated-fee-total`) at today's price, for a margin check on each SKU? It depends on the seller's VAT position (see the VAT section of [SKILL.md](../SKILL.md)):
  - **VAT-registered on standard accounting** (not on the Flat Rate Scheme): take the VAT out of `your-price` first (1/6 at 20%): `your-price` minus `estimated-fee-total` overstates the margin by the VAT inside the price. The VAT Amazon adds to its fees is reclaimed, so `estimated-fee-total` is the fee cost.
  - **Flat Rate Scheme:** the seller can't reclaim the VAT Amazon adds to its fees ([GOV.UK](https://www.gov.uk/vat-flat-rate-scheme/how-it-works)), so it is a cost: add it to `estimated-fee-total` (20% in the UK). For a rough VAT cost per sale, use the seller's flat rate times the VAT-inclusive `your-price` ([GOV.UK](https://www.gov.uk/vat-flat-rate-scheme/how-much-you-pay)), not 1/6 of it: ask them their rate, and label the margin an estimate.
  - **Not VAT-registered:** use `your-price` as it is. The seller can't reclaim the VAT Amazon adds to its fees, so it is a cost: add it to `estimated-fee-total` (20% in the UK).
  - **Whatever the VAT position,** `estimated-fee-total` leaves out Amazon's Digital Services Fee, a separate charge on its selling and FBA fees that shows as `DigitalServicesFee` in [`transactions`](transactions.md). Where it applies, add it for the full per-sale cost, taking its rate from the seller's recent `transactions` or Amazon's fee pages.
- Did a size tier, weight band or dimension change push a SKU's fulfilment fee up? Compare weekly snapshots.

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Can be UK only, and roughly the stocked SKUs.** The file can hold only `GB` rows (`amazon-store`) even when the request names every store. The SKUs roughly follow those with fulfillable stock in `inventory_all` the same day, but not exactly: the file can hold a SKU with none and miss a SKU that has some.
- **Estimated vs charged fees.** For fees actually charged, use `seller_economics`, whose fees include VAT: compare the estimate with its `amount` divided by `quantity`, not with `amountPerUnit` or `totalAmount`, which include the VAT. `amount` is before VAT and before fee promotions: the UK `FbaFulfilmentFee` `amount` per unit equalled this report's fulfilment fee on every SKU checked, including ones with a promotion. What was actually charged before VAT is `amount` minus `promotionAmount`, and a promotion can be the whole fee (`totalAmount` 0).
- **The EFN columns don't mean cross-border cost.** The DE, FR, IT, ES and SE columns are filled and the `-uk` column is `--` on every row. Their meaning isn't documented and all five filled columns equal the domestic fee on every row, so don't use them for the cost of a cross-border order.
- **HTML entities in SKUs:** `&pound;`, `&amp;` and `&ndash;`. Decode them before joining to other reports.
- Lines end in CRLF, but the last line of the file has no line ending at all, so `wc -l` counts one row short and a shell `while read` loop skips that row. The text is UTF-8.

Sources: [SP-API FBA reports](https://developer-docs.amazon/sp-api/docs/report-type-values-fba) (column names and which stores get them) and [Fee Preview Report](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201115050) (meanings).
