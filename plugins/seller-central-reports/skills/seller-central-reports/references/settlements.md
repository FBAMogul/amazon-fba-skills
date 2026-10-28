# Settlement reports (`settlements`)

Amazon's settlement reports (Flat File V2): one file per settlement period, listing every order, refund, fee and other amount that made up that payout.

## Where to find it in Seller Central

Checked in Seller Central UK on 28 Oct 2026: **Finance > Finance reports**, which opens **Payment reports**, then the **All Statements** tab. It lists each settlement period (about two weeks) with its beginning balance, sales, refunds, expenses, others and payout amount, and a download for each. The open period shows it is available only after the settlement closes. The From and To filter boxes take DD/MM/YYYY. The download button is **Download Flat File V2**, with **Download XML** and **Download Flat File** (the old flat file) in its drop-down.

Amazon's help calls this the All Statements page of the Payments Report ([View All Statements](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200801160); [Download payment reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G791)). It says a payout period is 14 days and the report can be downloaded once the settlement closes ([View All Statements](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200801160)).

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `getReports` with `reportTypes` = `GET_V2_SETTLEMENT_REPORT_DATA_FLAT_FILE_V2` and `processingStatuses` = `DONE` (`pageSize` up to 100). With no `createdSince`, Amazon's default window applies. Later pages send only `nextToken`, as Amazon requires ([getReports](https://developer-docs.amazon/sp-api/reference/getreports)). For each listed report, call `getReportDocument` with its `reportDocumentId`, then download the file (unzip it if Amazon sent it as GZIP). Amazon builds settlement reports itself; they "cannot be requested or scheduled" ([Settlement Reports](https://developer-docs.amazon/sp-api/docs/report-type-values-settlement)), so there is no `createReport` step. No report options.
- **Role:** [Finance and Accounting](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types).
- **How far back Amazon goes:** about 90 days. `createdSince` defaults to 90 days ago and "Reports are retained for a maximum of 90 days" ([getReports](https://developer-docs.amazon/sp-api/reference/getreports)), so download each settlement while it is listed and keep your own copy. In Seller Central, older statement periods have a **Request Report** button, and generated downloads stay for 90 days; how far back that goes is not stated ([View All Statements](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200801160)).

## Columns

Tab-separated text, header row first. The V2 flat file replaces the old flat file and XML versions, and puts every price and fee into three columns: `amount-type`, `amount-description` and `amount` ([Settlement Reports](https://developer-docs.amazon/sp-api/docs/report-type-values-settlement)).

**The first data row is a summary row.** It alone holds `settlement-start-date`, `settlement-end-date`, `deposit-date`, `total-amount` and `currency`; every other row leaves them blank and has the transaction columns instead.

| Column | Meaning | Format and traps |
|---|---|---|
| `settlement-id` | Amazon's unique id for the settlement | On every row. A number. Not the same as the report id that `getReports` lists. |
| `settlement-start-date` | Start of the settlement period | Summary row only. **`DD.MM.YYYY HH:MM:SS UTC`**, not ISO 8601. |
| `settlement-end-date` | End of the settlement period | Summary row only. Same format. |
| `deposit-date` | Date the payout is sent to the seller's bank account | Summary row only. Same format. |
| `total-amount` | Amount to be paid into the seller's bank account | Summary row only. Can be negative: a balance owed to Amazon, carried into that store's next settlement (see the EUR trap below). |
| `currency` | Currency of the settlement | Summary row only. One currency per file: `GBP` for the UK, `EUR` for euro stores. |
| `transaction-type` | Text description of the transaction type | Values include: `Order`, `Refund`, `other-transaction`, `FBAFees`, `AmazonFees`, `Order_Retrocharge`. |
| `order-id` | Amazon's order id | |
| `merchant-order-id` | Order id that Amazon or the seller assigns | |
| `adjustment-id` | Transaction id for adjustments | Filled on `Refund` rows and on `REVERSAL_REIMBURSEMENT` rows (`other-transaction`). A single space on `Order_Retrocharge` rows; blank on the rest. |
| `shipment-id` | "Order transaction type ID" (Amazon's wording) | Seen as 9 letters and digits. |
| `marketplace-name` | Store where the order was placed | e.g. `Amazon.co.uk`, or a euro store's name. One store per file. **Blank on every `other-transaction` row** (reimbursements, inbound transport fees, the subscription fee, and the EUR `Payable to Amazon` and `Successful charge` rows), so filtering on it drops them. |
| `amount-type` | Group the charge or fee belongs to | e.g. `ItemPrice`, `ItemFees`, `Promotion`, `ItemWithheldTax`, `other-transaction`, `FBA Inventory Reimbursement`, `FBA Inventory Storage Fee`. **EPR (extended producer responsibility) rows put their long text here**, in the pattern `Eco-contribution for EPR Pay on Behalf - <country> - <category> - (Period …)` or `Amazon service fee for EPR Pay on Behalf - …`, where `<country>` is a country code and `<category>` the EPR category the fee is for. Search this column for EPR, not `amount-description`. |
| `amount-description` | The specific charge or fee within that group | e.g. `Principal`, `Tax`, `Commission`, `DigitalServicesFee`, `FBAPerUnitFulfillmentFee`, `Shipping`, `RefundCommission`, `Base fee`, `Tax on fee`, `Payable to Amazon`, `Successful charge`. EPR rows have only `Base fee` or `Tax on fee` here. Both `DigitalServicesFee` and `Digital Services Fee` occur. Where Amazon calculates the VAT (its VAT Calculation Service), `Principal` is the price net of VAT and the `Tax` row beside it is the VAT. Without that, `Principal` can include VAT with no `Tax` row ([A2X's settlement examples](https://support.a2xaccounting.com/en/articles/3431567-getting-started-with-vat-in-a2x)): then don't treat it as net, and use the seller's accounting records for the VAT. The `Tax` row is 20% of `Principal`, within a penny, on a UK order item at the standard rate; where it isn't, look for an export where Amazon keeps the VAT, below. `ItemFees` rows, unlike `Principal`, include VAT (below). `TaxDiscount` is the VAT on a promotion's discount (seen on delivery promotions), while the `Promotion` / `Shipping` row beside it is net. Where the promotion makes delivery free, it equals minus the item's `ShippingTax`; on a smaller discount it's only the VAT on the discount, so use its own signed amount, never minus `ShippingTax`. Count it with the VAT, not with the promotion. |
| `amount` | The amount for that charge or fee | Decimal point in GBP files. **EUR files mostly use a decimal comma** (`-12,34`), but some storage and EPR rows in EUR files use a decimal point. No EUR amount has a thousands separator, so change `,` to `.` and never strip `.` (that makes the decimal-point amounts 100 times too big). |
| `fulfillment-id` | Fulfilment type | e.g. `AFN`; blank, or `Amazon` on storage fee rows (`FBAFees`). |
| `posted-date` | Date the item was debited or credited to the account | `DD.MM.YYYY`. |
| `posted-date-time` | Not defined; the same moment with the time | `DD.MM.YYYY HH:MM:SS UTC`. |
| `order-item-code` | Amazon's id for an item within the order | 14 digits. Usually matches `order-item-id` in [`orders`](orders.md). Where it doesn't, join on `order-id` plus `sku`, as for the [`transactions`](transactions.md) items that don't match. |
| `merchant-order-item-id` | Not defined | Always blank. |
| `merchant-adjustment-item-id` | Not defined | Filled on `Refund` rows. |
| `sku` | The seller's SKU | Special characters come through as plain text (a `£` is `£`, not `&pound;`). |
| `quantity-purchased` | Number of units purchased | Filled on every `Order` row (tax, fee and promotion rows too) and on some `other-transaction` rows; blank on refunds. Count units on `ItemPrice` / `Principal` rows only: summing every row counts each unit several times. |
| `promotion-id` | Seller-specified promotion id | Always blank. |

Every column Amazon lists is in the file. Amazon's help says the fields shown depend on the selling plan.

## Downloaded from Seller Central

**The same file as the API's.** Seen in a real UK seller's Seller Central download (Oct 2026): **Download Flat File V2** gives the same file as `GET_V2_SETTLEMENT_REPORT_DATA_FLAT_FILE_V2` through the API. It is tab-separated, with no byte order mark, lines ending in LF, and the same 24 columns in the same order. Everything under Columns and Traps applies. The file is named by a number, not the period, so rename it when you save it.

**Download XML** and **Download Flat File** are the older formats the V2 file replaced (see Columns). They weren't checked; use Flat File V2. For traps shared by all Seller Central downloads, see [seller-central-downloads](seller-central-downloads.md).

## Personal data

None of the columns hold buyer or business personal details. Some sellers put their cost price in the SKU, so treat SKUs as private.

## Questions it answers

- What made up each payout, and when did it reach the bank (`deposit-date`, `total-amount`)? A negative EUR total is owed to Amazon, not paid out.
- How much went on referral fees and FBA fulfilment fees in each settlement (`amount-type` `ItemFees`: `Commission`, `FBAPerUnitFulfillmentFee`, VAT included: see below), and how much on storage (`transaction-type` `FBAFees`, `amount-type` `FBA Inventory Storage Fee` and `FBA Long Term Storage Fee`, each as a `Base fee` and a `Tax on fee` row)?
- Which orders were refunded in a period, how much referral fee came back (`Commission`, positive, on `Refund` rows), and what did Amazon keep as its refund administration fee (`RefundCommission`, negative: the lesser of £5 or 20% of the referral fee, per [Amazon's UK pricing page](https://sell.amazon.co.uk/pricing))? Both rows include VAT, so the £5 cap shows here as £6.00.
- Did Amazon hold back a reserve? Look for a line that mentions a reserve.
- Does a bank deposit match the orders and fees behind it, store by store? Each file is one store, so reconcile file by file, not by `marketplace-name` (blank on `other-transaction` rows).

## Traps seen in real data

Seen in a real UK seller's files (Oct 2026):

- **Amazon still listed settlements a little older than 90 days.** Don't rely on it: download each settlement while it is listed.
- **Two kinds of file.** UK files (`GBP`, `Amazon.co.uk`) hold the UK orders. A UK seller with stock or fees in EU stores can also get EUR files, one per store each settlement period. A store with no sales in the period still gets a file, holding only its fees and any carried balance.
- **EUR files carry a balance forward.** Each has one `other-transaction` row `Payable to Amazon`, which equals the same store's previous `total-amount`: the unpaid balance. Some also have a positive `Successful charge` row, a payment against the balance. Leave both row types out of fee and cost sums: adding them in can overstate a small store's fees many times over. Keep them when reproducing `total-amount`: all the rows together add up to it.
- **Amazon keeps the VAT on some exports.** On UK order items delivered abroad (by `ship-country` in [`orders`](orders.md)), an `ItemWithheldTax` / `MarketplaceFacilitatorVAT-Principal` row equals minus the `Tax` row. These are the items whose `Tax` isn't 20%. So for a seller on standard VAT accounting, `Tax` overstates the VAT they owe, and for any seller `Principal` plus `Tax` overstates what they received, unless the `ItemWithheldTax` rows are counted too.
- **Fee rows include VAT; `Principal` doesn't, where there's a `Tax` row** (see `amount-description`). Each `ItemFees` row is the fee with its VAT in one amount, with no tax row beside it. Joined to [`transactions`](transactions.md) on `order-item-code`, `Commission` equals that item's `Commission` `Base` plus `Tax` (with `Tax` 20% of `Base`), and `FBAPerUnitFulfillmentFee` equals its fee total there (`Base` plus `Tax`, and sometimes a fee discount, `Promo`). The fulfilment fee can carry no VAT, mostly on items delivered abroad. The other `ItemFees` rows, and the fee rows on refunds (joined on `order-item-code` and `adjustment-id` = `REFUND_ID`), also equal `Base` plus `Tax`. So net `Principal` minus these fees counts the fees' VAT as a cost. That suits a seller who isn't VAT-registered, who can't reclaim fee VAT. A Flat Rate Scheme seller can't reclaim it either, but their sales start VAT-inclusive (`Principal` plus its `Tax`), less the VAT they owe as their VAT return or accountant gives it. A seller who reclaims fee VAT (VAT-registered on standard accounting) has the fee before VAT as their fee cost. See the VAT section of [SKILL.md](../SKILL.md). And [`fba_fee_preview`](fba_fee_preview.md)'s estimates, which leave VAT out, show a false 20% overcharge beside these rows. For fees before VAT, take each fee's total in `transactions` minus its `Tax`, which is `Base` plus any `Promo`. The fulfilment fee can have a `Promo` (a fee discount, positive), which can waive the whole fee, so summing `Base` alone overstates fulfilment fees before VAT.
- **Settlements vary in length.** A settlement can be much shorter than 14 days, and there can be a gap of several days between one listed settlement's end and the next one's start. Use `transactions` for a period no settlement covers.
- **Dates are `DD.MM.YYYY`** with `UTC`, and EUR amounts usually use a decimal comma, while some EUR rows use a decimal point (see `amount`). Convert both before sorting or summing.
- **The first row is the summary.** Skip it when adding up transactions.
- Lines end in LF, not CRLF.

Sources: column list and the three amount columns from [Settlement Reports](https://developer-docs.amazon/sp-api/docs/report-type-values-settlement); meanings and the `amount-type`/`amount-description` table from [Download payment reports](https://sellercentral.amazon.co.uk/help/hub/reference/external/G791).
