---
name: seller-central-reports
description: Use when the user asks about their Amazon Seller Central reports or data - orders, sales, traffic, fees, settlements, transactions, inventory, stock movements, returns, reimbursements, removals, listings, account health or Business Reports - when reading a report file downloaded from Seller Central or pulled through the Selling Partner API (SP-API), or when building a dashboard or spreadsheet from Amazon seller data. Says where each report is in Seller Central, which API call and role gets it, how far back Amazon goes, what every column means, and the traps in the data, for UK sellers.
---

# Seller Central reports

This pack describes the Amazon Seller Central reports a UK FBA seller can download or pull through the Selling Partner API (SP-API): where each one is, how to get it, what every column means, and the traps in the data. It was researched in October 2026 from Amazon's own pages and checked against a real UK seller's report files, and its menu paths and Seller Central downloads were checked in Seller Central UK on 28 Oct 2026. Amazon changes its reports, menus and rules without much notice, so treat a page as a strong starting point, not the last word.

UK accounts are in Amazon's EU region (endpoint `sellingpartnerapi-eu.amazon.com`, UK marketplace ID `A1F83G8C2ARO7P`), and a UK account often takes part in several EU stores as well as amazon.co.uk.

## Ground rules

- **Never make up a number.** If a figure isn't in the file, leave it blank and say it isn't there. Don't estimate, fill in or carry a figure over from another month unless the seller asks for an estimate, and then label it as one.
- **Check before a money or stock decision.** Before the seller uses a figure for a money or stock decision (a reimbursement claim, a reorder, a removal, a tax or profit figure), check it against the file itself: re-read the rows it came from and show how it was worked out. Every review of these pages has found new traps.
- **Strip buyer personal data first.** Before reading any rows of a file that holds personal data, write and run a small script on the seller's computer that copies the file without those columns (found by name in the header row, not by position; for a JSON file, without those fields). Work only from the cleaned copy. Never print, `head` or `cat` the original, or put any of its rows into the chat; the seller keeps the original privately. Never copy personal details into anything shared (a spreadsheet for a VA, a screenshot, a forum post, a public dashboard). Each report page's **Personal data** section names the columns:
  - buyer city, county and postcode in `orders`, `fba_shipments` and `fba_shipment_sales`, and `order city`, `order state` and `order postal` in `date_range_transactions`
  - buyer emails in `fba_shipments`
  - in `vat_transactions`, the buyer's name, full delivery address, city, postcode and VAT number, the departure city and postcode (the buyer's location on return rows), and the seller's own VAT numbers
  - buyers' own words in `fba_returns` and `seller_feedback`, and the seller's public replies in `seller_feedback` (a reply can quote the buyer)
  - the seller's own business and contact details in `account`, `inbound_plans` and `store_list` (`storeName`)

  Amazon's [Data Protection Policy](https://sellercentral.amazon.com/mws/static/policy?documentType=DPP&locale=en_GB) covers information from "Amazon Portals" as well as the API, so treat Seller Central downloads like API data. It says buyer personal data is kept "no longer than 30 days after order delivery" unless a law requires it, so any copy the seller saves should drop the buyer columns.
- **Keep the seller's figures and SKUs private.** Sales, profit, fees and stock figures are the seller's business. Some sellers put their cost price in the SKU, so treat SKUs as private too.
- **API use stays read-only.** Request reports and read data only. Never call anything that changes the account: prices, listings, shipments, cases or messages (for example Listings Items or Feeds writes, price changes, inbound shipment creation, the Messaging API), even when the app's roles would allow it. A bug in a write call changes the live account.
- **Never touch API keys.** Never ask for, print, store or paste the client ID, client secret or refresh token. They live in a `.env` file that scripts read when they run; Claude doesn't open it (see [getting-access](references/getting-access.md)).

## Where things are in Seller Central

UK accounts now open the new Seller Central, whose ☰ menu has My business, Products, Supply chain, Orders, Marketing, Finance, Account health and Programs and apps. The FBA reports sit under **Supply chain > Supply chain reports**, order and return reports under **Orders > Order reports**, payment reports under **Finance > Finance reports**, and listing reports under **Products > Product reports**. Some pages, Business Reports among them, are only in the **Site map** link at the bottom of the menu. The old layout is still there as **Classic Seller Central**, but Amazon says it is going. Each report page gives both paths; [seller-central-map](references/seller-central-map.md) has the full menu.

## Reading report files

**The column notes describe the files as the API sends them.** Most Seller Central downloads are the same report, but some aren't: Business Reports, SKU Economics, the Date Range transaction report and Transaction View give different files with different column names, and the same report can come as CSV or tab-separated text with a different date style. Each report page's **Downloaded from Seller Central** section, where it has one, says how the two compare; [seller-central-downloads](references/seller-central-downloads.md) has the traps they share. If a column isn't where the page says, say so, and find it by name rather than position.

### SKUs

- **HTML entities.** Some reports write `£` in a SKU as `&pound;`: `reimbursements`, `fba_returns`, `inventory_by_country`, `reserved_inventory`, `stranded`, `recommended_removals`, `removal_orders`, `removal_shipments` and `fba_fee_preview`. Others write `£`: `orders`, `fba_shipments`, `fba_shipment_sales`, `ledger_detail`, `ledger_summary`, `inventory_all`, `inventory_planning`, `restock`, `listings_all`, `pan_eu_status`, `long_term_storage_fees`, `settlements`, `seller_economics` and `transactions`. Some reports also write `&amp;`, `&quot;` or `&ndash;`. Decode every HTML entity, not just `&pound;`, before joining SKUs; replacing `&pound;` alone leaves some SKUs unmatched.
- **Backslashes.** `transactions` puts a backslash before a `"` inside a SKU, even after JSON parsing. Remove it before joining on SKU.
- Amazon-made SKUs look like `XX-XXXX-XXXX`, and `amzn.gr.…` SKUs are Amazon's Grade and Resell units.

### Quoting and line endings

- **Ledger files.** `ledger_detail` and `ledger_summary` wrap every header and value in double quotes, except a blank `Reason` in `ledger_detail`, which is empty with no quotes. Quotes inside a value aren't doubled (some SKUs and titles hold a bare `"`, and a SKU can end in one). So split each line on tab and take off only the outer pair of quotes. **Never use a quote-aware CSV reader on these files:** Python's `csv` silently shifts columns, and Ruby's `CSV` raises an error.
- `storage_fees` wraps `fulfillment_center` in single quotes.
- `listings_all`, `suppressed_listings` and `pan_eu_status` start with a UTF-8 byte order mark.
- Most FBA reports end lines in CRLF. Most tab-separated reports leave the last line with no line ending (not `orders`, `listings_all`, `suppressed_listings`, `fbm_returns` or `settlements`), so `wc -l` counts one row short and a shell `while read` loop skips the last row. Count rows with a proper parser.
- Find columns by name: `listings_all` headers differ between stores.

### Dates and decimals

- Most reports use ISO 8601. `ledger_detail`, `stranded` and `restock` use MM/DD/YYYY; `ledger_summary` uses MM/YYYY; `listings_all` uses DD/MM/YYYY with a zone name; `settlements` uses DD.MM.YYYY with `UTC`.
- `removal_orders` and `removal_shipments` change time zone offset from row to row, so convert to one zone before comparing.
- Most EUR settlement amounts use a decimal comma, but some rows in the same files use a decimal point. Change `,` to `.` and never strip `.`.

### VAT: included or not, per file

**First find out the seller's VAT position: registered or not, and if registered, on the Flat Rate Scheme or not.** Ask them. A UK seller who isn't registered should see no VAT in `orders`' `item-tax` on UK orders, but confirm by asking rather than relying on that.

- **Standard VAT accounting** (VAT-registered, not on the Flat Rate Scheme): the steps in this section for taking VAT out of prices and fees are for them. Most UK sales carry 20% VAT, which is 1/6 of a VAT-inclusive price. They reclaim the VAT Amazon charges on its fees ([GOV.UK](https://www.gov.uk/charge-reclaim-record-vat/reclaim-vat-business-expenses): "You can reclaim VAT on items you buy for use in your business if you're VAT registered"), so the fees before VAT are the cost.
- **Flat Rate Scheme:** they can't reclaim the VAT on Amazon's fees ([GOV.UK](https://www.gov.uk/vat-flat-rate-scheme/how-it-works): "you cannot reclaim the VAT on your purchases - except for certain capital assets over £2,000"), so use fees including VAT. The VAT they owe isn't the VAT inside each price: it's their flat rate times their VAT-inclusive turnover ([GOV.UK](https://www.gov.uk/vat-flat-rate-scheme/how-much-you-pay): "You calculate the tax you pay by multiplying your VAT flat rate by your 'VAT inclusive turnover'"), and HMRC has its own rules on what counts in that turnover, such as refunds, exports and goods sold from abroad ([VAT Notice 733, section 6](https://www.gov.uk/guidance/flat-rate-scheme-for-small-businesses-vat-notice-733--2)). So don't take 1/6 out of prices, and don't work out the VAT owed from Amazon's reports: keep sales with VAT, and take the VAT owed from the seller's VAT return or their accountant. If the seller wants a rough figure, multiply their flat rate by their VAT-inclusive sales and label it an estimate.
- **Not VAT-registered:** don't take VAT out of prices, as there is none to pay over. The VAT Amazon charges on its fees is a real cost they can't reclaim (only VAT-registered businesses can, as the GOV.UK page above says), so use fees including VAT.

Whatever the position, put both sides on the same basis before joining or comparing. The table says which figures include VAT.

| Report and column | VAT |
|---|---|
| `orders` `item-price` | Includes VAT; `item-tax` is the VAT inside it. |
| `orders` `item-promotion-discount` | Before VAT; `item-tax` on those rows is the VAT after the promotion, so the buyer paid `item-price` minus the discount with its VAT at the item's VAT rate (− 1.2 × `item-promotion-discount` on a 20% item). Don't work the rate out from `item-tax` on a promotion row: a standard-rated item made free by a promotion shows `item-tax` `0` (see [orders](references/orders.md)). |
| `fba_shipments` `item-price`, `fba_shipment_sales` `item-price-per-unit` | Before VAT for a seller enrolled in Amazon's VAT Calculation Service, with `fba_shipments` adding `item-tax` on top; Amazon says enrolling changes them from VAT-inclusive ([Amazon](https://developer-docs.amazon/sp-api/docs/create-and-upload-invoices)). Check which: before VAT, `item-price` + `item-tax` equals `orders`' VAT-inclusive `item-price` for an item with one row. |
| `settlements` `Principal`, `transactions` `ProductCharges` | Before VAT, with the VAT on a separate `Tax` row or breakdown, where Amazon calculates the VAT (its VAT Calculation Service). Without that, `Principal` can include VAT with no `Tax` row ([A2X's settlement examples](https://support.a2xaccounting.com/en/articles/3431567-getting-started-with-vat-in-a2x)). If there's no `Tax` row, don't treat the price as before VAT: ask the seller and use their accounting records. |
| `settlements` fee rows (`ItemFees`: `Commission`, `FBAPerUnitFulfillmentFee` and the rest) | Include VAT. Each equals that fee's total in `transactions`. |
| `transactions` fees | Before VAT = the fee's total minus its `Tax` (`Base` plus any `Promo`). `Base` alone overstates a fee that a `Promo` partly waived, as fulfilment fees sometimes are. |
| `fba_fee_preview` | `your-price` includes VAT; the fee estimates leave out VAT and fee promotions, so they look 20% low beside `settlements`. On standard VAT accounting, take the VAT out of the price before taking off the fees. On the Flat Rate Scheme, keep the price as it is, add the fees' 20% VAT, and for a rough VAT cost take their flat rate of the price off, labelling the margin an estimate. If not registered, keep the price as it is and add the fees' 20% VAT. |
| `removal_orders` `removal-fee`; `transactions` `FBARemoval` and `FBADisposal` `totalAmount` | Include VAT. Before VAT = those charges' `Base` plus `Promo`. |
| `storage_fees`, `long_term_storage_fees` | Before VAT. |
| `sales_and_traffic`, `seller_analytics`, `seller_analytics_asin` sales | Include VAT, like `orders` `item-price`. Their deal prices (`dealOurPrice`, `promotionOrderedPrice`) look to leave it out. |

- **Withheld VAT.** On some exports Amazon keeps the VAT and cancels it with a `MarketplaceFacilitatorVAT-…` line (in `settlements` an `ItemWithheldTax` row; in `transactions` a line under `Tax`). Take VAT from `transactions`' `Tax` total or count the `ItemWithheldTax` rows, never `OurPriceTax` or the `Tax` row alone.
- **`seller_economics` changed basis.** Seen in a real UK seller's data (Oct 2026): `orderedProductSales` includes VAT up to about 24 Aug 2025 and leaves it out from about 25 Aug 2025, a false drop of about 1/6 on 20% items. `refundedProductSales` leaves VAT out in every month, so before the change `netProductSales` is sales with VAT minus refunds without it. Fees carry their VAT, where they have any, in `taxAmount` throughout, so `netProceeds` mixes bases. For a seller who reclaims fee VAT (standard VAT accounting), the figure before VAT is `netProceeds` with every fee's `taxAmount` added back and, for months before the change, the VAT inside `orderedProductSales` taken off (not the VAT on `netProductSales`, which takes off too little, as refunds never held VAT). A Flat Rate Scheme or non-registered seller can't reclaim fee VAT, so don't add it back; [seller_economics](references/seller_economics.md) says how a Flat Rate seller treats the sales. Don't mix Business Reports or Seller Analytics sales with `seller_economics` from late August 2025 on.
- **Comparing fee estimates.** Compare `fba_fee_preview`'s estimates with `seller_economics`' `amount` divided by `quantity`, not its `amountPerUnit` or `totalAmount`. What was actually charged before VAT is `amount` minus `promotionAmount`.
- Seen in a real UK seller's data: some older `seller_economics` SKU-months (up to January 2026) had doubled units and refunds. See its page.

### Join keys

- **Order items:** `order-item-id` (`orders`) = `amazon-order-item-id` (`fba_shipments`) = `order-item-code` (`settlements`) = `ORDER_ADJUSTMENT_ITEM_ID` (`transactions`). Three catches:
  - `fba_shipments` can hold several rows for one item, sometimes across more than one shipment, so never sum `orders`' price or quantity per joined row.
  - Check both files cover the same orders. `transactions` lines for orders placed before the `orders` file starts have nothing to join to; leave them out when comparing the two.
  - Some `transactions` `Shipment` and `Refund` items, and now and then a `settlements` `Order` item, carry an item id that is in neither `orders` nor `fba_shipments`. Join those on the order id (`ORDER_ID`, `order-id`) plus SKU, after dropping `orders` rows whose `item-status` is `Cancelled`. Where the `orders` row has a different SKU for the same ASIN, order id plus ASIN matches. Where an order has two `Shipped` rows for the SKU, match `quantityShipped` to `quantity`. One `orders` row can match several items, so here too never sum its price or quantity per joined row.
- **FBA reports:** FNSKU joins the other FBA reports (`fba_shipments` has none). `seller_economics` has no usable FNSKU, so join it on SKU.

### Which stock file to trust

- **UK stock ready to pick:** `inventory_all`.
- **EU stock:** `inventory_by_country`'s non-GB rows. Stock held in an EU centre can be billed for storage but missing from the stock reports, so cross-check `storage_fees` against the stock files.
- **Stranded units** are already in `inventory_by_country`'s GB rows; `inventory_all` leaves them out.
- `restock` and `inventory_planning` can lag `inventory_all` by minutes to a day.
- **Never add inbound from one file to stock from another.**
- **All UK stock in the centres:** `inventory_all`, plus the stranded units (from `stranded`), plus any UK FNSKUs that `inventory_all`, `restock` and `inventory_by_country` all leave out, taking their units from `inventory_planning`. These can be FNSKUs holding only reserved units (also listed in `reserved_inventory`) or only unsellable units.

## Before you trust a number

- **Recent months still change.** Returns, reimbursements and payments land after the sale, Seller Analytics figures fill in over the following weeks, and removal orders and shipments can change months after the month they were requested in. A report pulled while a month is still running holds only part of it. Download a recent month again before relying on it, and say how recent the file is.
- **History has limits.** Each report page gives Amazon's limit (also in the index below). Snapshot reports (inventory, listings, account health, fee preview) show only how things stand when asked: Amazon keeps no past copies, so history exists only if the seller saved one each time. The API keeps settlement reports for about 90 days.
- **A cancelled report isn't always an error.** Amazon cancels a report when there's no data for the period, or when the request falls inside the report's once-per-period limit ([Reports FAQ](https://developer-docs.amazon/sp-api/docs/reports-faq)). Some reports send a header-only file instead.
- **Throttling Amazon doesn't document.** Amazon's documented limit for report requests is about one a minute after a burst of 15 (`createReport`: 0.0167 requests a second, burst 15, in [Amazon's Reports API model](https://github.com/amzn/selling-partner-api-models/blob/main/models/reports-api-model/reports_2021-06-30.json)). In practice, after a few quick requests for the same report type, Amazon can return HTTP 429 for that type and keep refusing it for a long while, while still accepting other types. The storage fee report looked to have a small daily limit: if it's refused, wait a day. `fba_shipments` has a documented limit of 12 requests per 8 hours, and Seller Central downloads count towards it.

## Report pages

One page per report in `references/`, each with where to find it, the API call and role, how far back Amazon goes, every column, personal data, the questions it answers and the traps seen in real data. "Amazon:" gives Amazon's history limit.

**Orders and sales**
- [orders](references/orders.md): the All Orders report: every order placed, FBA and seller-fulfilled, one row per order item, by purchase date. Amazon: 2 years (older orders only through Archived Order Reports), at most 30 days per request.
- [fba_shipments](references/fba_shipments.md): every completed FBA shipment to a buyer, item by item, with prices, carrier, tracking and fulfilment centre. Amazon: 18 months, at most 31 days per request.
- [fba_shipment_sales](references/fba_shipment_sales.md): shipped FBA orders, one row per item shipped, with unit price, FNSKU and fulfilment centre. Amazon: not stated for the API; Amazon's US help says customer order data is kept 18 months.
- [sales_and_traffic](references/sales_and_traffic.md): Business Reports sales and traffic (page views, sessions, Buy Box %) for one store, by day and per child ASIN (JSON). Amazon: 2 years.
- [seller_analytics](references/seller_analytics.md): Data Kiosk Seller Analytics: monthly sales, traffic, promotions, reviews, returns and account health per store, plus account totals. Amazon: 2 years; with its snapshot metrics, the current month and the previous 23.
- [seller_analytics_asin](references/seller_analytics_asin.md): the same Seller Analytics metrics per ASIN per store, by month. Amazon: 2 years.
- [seller_feedback](references/seller_feedback.md): neutral and negative (1 to 3 star) seller feedback, with the comment and any reply. Amazon: not stated for the API; a Seller Central report covers up to one year.
- [fbm_returns](references/fbm_returns.md): returns of seller-fulfilled (FBM) orders. Amazon: not stated; at most 60 days per request.

**Money**
- [settlements](references/settlements.md): settlement reports (Flat File V2), one per payout period, with every order, refund, fee and other amount in it. Amazon: about 90 days through the API; older periods can be requested again in Seller Central.
- [transactions](references/transactions.md): every financial transaction (Finances API): order payments, refunds, fees, transfers, with the fee breakdown and deferred or released status. Amazon: not stated; at most 180 days per request; Seller Central's Transaction View shows 24 months.
- [seller_economics](references/seller_economics.md): Data Kiosk Seller Economics: sales, every fee by type, ad spend, the seller's entered costs and net proceeds per SKU per store, by month. Amazon: 2 years.
- [date_range_transactions](references/date_range_transactions.md): the Date Range transaction report from Seller Central's Reports Repository (CSV): every posted transaction with its fees, released or deferred. Seller Central only; the API route to the same data is `transactions`.
- [vat_transactions](references/vat_transactions.md): the Amazon VAT Transactions Report: every sale, refund and Amazon fee with VAT amounts, rates and departure and arrival countries, one month per file.

**Stock movements and FBA events**
- [ledger_detail](references/ledger_detail.md): every FBA stock movement: customer shipments, receipts, returns, removals, transfers, lost, found, damaged, disposed. Amazon: 18 months. Can leave out stock held in EU centres.
- [ledger_summary](references/ledger_summary.md): monthly stock balance per FNSKU, opening to closing, with the month's movements by type. Amazon: 18 months (Seller Central help). Can leave out stock held in EU centres.
- [reimbursements](references/reimbursements.md): FBA reimbursements, cash or replacement units, with reason, case ID and amounts, reversals included. Amazon: not stated for the API; Seller Central's search reaches back 18 months.
- [fba_returns](references/fba_returns.md): FBA customer returns with reason, condition and status. Amazon: not stated.
- [removal_orders](references/removal_orders.md): removal and disposal orders with quantities and the removal fee. Amazon: not stated.
- [removal_shipments](references/removal_shipments.md): shipments sent out for removal orders, with carrier and tracking. Amazon: not stated.
- [inbound_noncompliance](references/inbound_noncompliance.md): problems Amazon found when receiving FBA shipments, with any Unplanned Service fee. Amazon: not stated for the API; Seller Central's Inbound Performance keeps 365 days.
- [inbound_plans](references/inbound_plans.md): FBA inbound plans (Send to Amazon), one summary per plan. Amazon: current list only.

**Inventory snapshots** (each shows stock as it stands when asked; Amazon keeps no past copies)
- [inventory_all](references/inventory_all.md): Manage Inventory: FBA stock per SKU by status, archived listings included.
- [inventory_by_country](references/inventory_by_country.md): FBA stock per SKU per European country.
- [reserved_inventory](references/reserved_inventory.md): reserved units per SKU, by reason.
- [restock](references/restock.md): restock recommendations per SKU and country.
- [stranded](references/stranded.md): stock with no active offer, with the reason and the action needed.
- [inventory_planning](references/inventory_planning.md): inventory health and age: recent sales (up to 90 days), aged and excess units, estimated fees.
- [recommended_removals](references/recommended_removals.md): removals Amazon recommends before the next clean-up. Check its `snapshot-date`: it has been seen months out of date.

**Fees**
- [storage_fees](references/storage_fees.md): estimated monthly storage fee per ASIN per fulfilment centre. Amazon: not stated.
- [long_term_storage_fees](references/long_term_storage_fees.md): aged inventory surcharge (long-term storage fees) per SKU, country and age tier, one month per request. Amazon: not stated; seen not to reach back past about 12 months (use `transactions` for older charges).
- [fba_fee_preview](references/fba_fee_preview.md): estimated referral and FBA fees per SKU with stock at Amazon. Snapshot; once a day.

**Listings**
- [listings_all](references/listings_all.md): every listing in one store, with status and fulfilment channel. Snapshot per store.
- [suppressed_listings](references/suppressed_listings.md): suppressed listings in one store, with the reason. Snapshot per store.
- [pan_eu_status](references/pan_eu_status.md): Pan-EU enrolment and per-store offer status per listing (amazon.co.uk left Pan-EU in 2021, but a UK seller can still enrol stock held in the EU stores). Snapshot.

**Account**
- [account_health](references/account_health.md): the Account Health dashboard (JSON): status, metrics against Amazon's targets, Account Health Rating, policy-violation counts. Snapshot; each metric has its own date range.
- [account_health_v1](references/account_health_v1.md): older customer metrics (XML), including contact response time. Snapshot.
- [store_list](references/store_list.md): the stores the account can list in and whether it takes part (JSON). Snapshot.
- [account](references/account.md): the seller account's business details (JSON): business type, selling plan, registered details, primary contact. Snapshot.

## More references

- [report-catalogue](references/report-catalogue.md): every SP-API report type, Data Kiosk, the roles and the rate limits.
- [seller-central-map](references/seller-central-map.md): the UK Seller Central menu (new and classic, checked 28 Oct 2026) mapped against the API.
- [seller-central-downloads](references/seller-central-downloads.md): downloading reports by hand from Seller Central: file names, formats, date styles and how each download compares with the API file, including Transaction View, fee invoice CSVs, Custom analytics and Voice of the Customer.
- [not-in-the-api](references/not-in-the-api.md): Seller Central pages with no API route (account health detail, messages, ungating, capacity and IPI, claims and more), and API data no page here covers.
- [getting-access](references/getting-access.md): how a seller gets their own SP-API access as a private developer, keeps it working, and keeps the keys safe.

**If API calls start failing:** the client secret must be renewed every 180 days, and the Solution Provider Portal's expiry warning has been unreliable (Oct 2026), so check the secret first. [getting-access](references/getting-access.md) has the common errors.
