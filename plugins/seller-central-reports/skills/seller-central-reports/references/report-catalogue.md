# Amazon SP-API + Amazon Ads API: what a UK FBA seller can read about their own account

Researched 18 Oct 2026; Amazon changes things, so check anything that matters.

This is the full map of the Selling Partner API (SP-API) for a seller, plus the Amazon Ads API. It was taken from Amazon's own docs (pulled as raw markdown and API model files, not from memory). Every claim carries a reference tag like **[orders]**. The full URL for each tag is in the **References** section at the end. Report types that have their own page in this pack link to it. For the Seller Central screens, and which of them have no API route, see [seller-central-map](seller-central-map.md).

- **Domain note:** `developer-docs.amazon.com` now 301-redirects to `developer-docs.amazon` (new `.amazon` domain). Both URLs work. The references give the URL that was actually read.
- **Ads docs note:** `advertising.amazon.com/API/docs` renders in JavaScript. Amazon's own `llms.txt` points to a raw-markdown mirror on CloudFront (`d3a0d0y2hgofx6.cloudfront.net/en-us/…`), and that mirror is what was read **[ads-llms]**.
- **"Unverified"** means Amazon's docs don't say, or it couldn't be confirmed from a primary source.

---

## 0. The short version

1. **A private app on your own account is free.** You register once as a *private developer* in the Solution Provider Portal (solutionproviderportal.amazon.com, or Classic Seller Central > Apps and Services → Develop Apps; the new Seller Central menu has no entry for it), self-authorise, and get a refresh token **[reg-priv] [selfauth]**. Amazon announced a $1,400/yr fee plus GET-call fees for 2026, then cancelled them. The cancellation page says Amazon "will not move forward with the SP-API usage and annual fees at this time" **[fees-cancel]**. Secondary reporting says sellers using the API only for their own business were never in scope **[ppcland]**.
2. **Nearly everything financial, inventory, order, listing and performance-related is readable with non-restricted roles.** The roles are Finance and Accounting, Amazon Fulfillment, Inventory and Order Tracking, Product Listing, Pricing, Selling Partner Insights and Brand Analytics **[roles] [rmap] [rtypes]**.
3. **The hard limits on history:**
   - Orders: 2 years **[orders]**.
   - Finances: 180-day windows, and group-level events only for the past 2 years **[fin-model] [finv0-model]**.
   - Settlement reports: Amazon auto-generates them and you can't request them. Reports are retained for at most **90 days**, so the API can't fetch settlement files older than about 90 days **[rt-settle] [reports-model]**.
   - Inventory ledger detail: 18 months **[rt-fba]**.
   - Sales & Traffic and Data Kiosk: 2 years **[rt-analytics] [dk-sa]**.
   - Ads API v1 reporting: 24 months of daily data and 6 years of monthly data **[ads-v1-dates]**.
4. **Account health is readable:**
   - [`GET_V2_SELLER_PERFORMANCE_REPORT`](account_health.md), a current snapshot of the Account Health dashboard **[rt-perf]**.
   - The `ACCOUNT_STATUS_CHANGED` notification **[notif]**.
   - The new (Sept 2026) Data Kiosk **Seller Analytics** dataset, which has account-health rates and counts by period going back up to 2 years **[dk-sa] [relnotes]**.
   - There is no separate "Account Health API" **[models]**.
5. **UK VAT reports need a restricted role.** `SC_VAT_TAX_REPORT`, `GET_VAT_TRANSACTION_DATA` and the VAT invoice data reports need **Tax Invoicing (Restricted)** plus a Restricted Data Token **[rt-tax] [rt-invoice]**. Buyer name and address on FBA orders also need Tax Invoicing or Tax Remittance (Restricted). The Orders API never gives buyer email or phone for FBA orders outside the US **[orders-pii]**. The exception is the Amazon Fulfilled Shipments report ([fba_shipments](fba_shipments.md)), whose `buyer-email` column can hold the buyer's email on UK orders.
6. **Brand-only data is out for a reseller who isn't the brand owner.** Brand Analytics search and market-basket reports need Brand Registry **[rt-analytics]**. Sponsored Brands needs Brand Registry **[ads-sb-elig]**.
7. **Deadlines to know:**
   - The old settlement report types (`…FLAT_FILE`, `…XML`) are removed on **11 Nov 2026**. Use [`GET_V2_SETTLEMENT_REPORT_DATA_FLAT_FILE_V2`](settlements.md).
   - Orders v0 is removed **27 Mar 2027**.
   - Finances v0 is removed **27 Aug 2027** **[deprec]**.
   - Ads reporting v3 shuts off after **30 Jun 2027** **[ads-rep]**.

---

## 1. Every SP-API section relevant to a seller

Roles are "need at least one of". "(R)" marks a restricted role. The role lists come from Amazon's operation-to-role mapping **[rmap]**. Seller-relevant APIs only; vendor-only APIs are listed at the end.

| API (current version) | What a seller can READ | Role(s) needed | UK / notes | Source |
|---|---|---|---|---|
| **Orders v2026-01-01** (`searchOrders`, `getOrder`) | All orders created or updated in a window, with items. `includedData` adds `BUYER`, `RECIPIENT`, `PROCEEDS`, `EXPENSE`, `PROMOTION`, `CANCELLATION`, `FULFILLMENT`, `PACKAGES`, `TAX`, `PAYMENT`, `FULFILLMENT_ORDERS`. | Any of: Amazon Fulfillment, Buyer Communication, Buyer Solicitation, Finance and Accounting, Inventory and Order Tracking, Pricing, Product Listing, Selling Partner Insights, or one of the restricted roles Direct to Consumer Shipping, Professional Services, Tax Invoicing or Tax Remittance (not Selling Partner Support). PII fields need restricted roles (§5). | Orders older than 2 years aren't returned. The new version needs no RDT for PII. | [orders] [orders-model] [orders-mig] [rmap] |
| Orders v0 (legacy) | Same data, spread across 6 operations (`getOrders`, `getOrderItems`, `getOrderBuyerInfo`, `getOrderAddress` …). | As above | **Deprecated 28 Jan 2026; removed 27 Mar 2027** | [deprec] |
| **Reports v2021-06-30** | All report types in §2 (request, schedule, list, download). | Depends on report type (§2) | Reports are retained for 90 days max | [reports-model] [rtypes] |
| **Data Kiosk v2023-11-15** | GraphQL datasets: Seller Analytics, Sales & Traffic, Seller Economics (§3). | Brand Analytics (to call the API); the Economics dataset also needs Selling Partner Insights. Field-level role checks also apply. | All regions. No server-side scheduling. | [dk] [rmap] |
| **Finances v2024-06-19** ([`listTransactions`](transactions.md), `listBalances`, `listSummary`) | Every posted transaction (deferred, released), balances (current or as of a past date), and financial summaries by period or settlement ID. | Finance and Accounting | `listBalances` and `listSummary` added in 2026. A request window over 180 days returns an empty response. The last 48 hours may be missing. | [fin] [fin-model] [relnotes] |
| Finances v0 (legacy) | Financial event groups (≈ settlements), events by group, by order, or by date. | Finance and Accounting (EU). AISP is also listed. | **Deprecated 21 Jul 2025; removed 27 Aug 2027**. `listFinancialEventsByGroupId` returns only the past 2 years. | [deprec] [finv0-model] |
| **Transfers v2024-06-01** | Payment methods, payouts (`listPayouts`), and expected payouts (`listExpectedPayouts`). Also initiates payouts. | `getPaymentMethods`: AISP, Finance and Accounting or PISP. `listPayouts` / `listExpectedPayouts`: Finance and Accounting. | `getPaymentMethods` is EU region only | [transfers] [rmap] [relnotes] |
| Seller Wallet v2024-03-01 | Seller Wallet accounts, balances, transactions, transfer schedules. | **AISP** (PISP to move money) | AISP and PISP are open-banking roles for **licensed third-party payment providers** (an FCA orgID is required). A normal seller app won't get them. | [wallet] [tpp] |
| **Sellers v1** | [`getMarketplaceParticipations`](store_list.md): stores you sell in, plus `hasSuspendedListings`. [`getAccount`](account.md): business name, address, company registration and tax ID, selling plan. | Participations: Product Listing or Selling Partner Insights. `getAccount`: Finance and Accounting or AISP. | — | [sellers] [sellers-model] [rmap] |
| **Sales v1** (`getOrderMetrics`) | Aggregated units, orders and sales by hour, day, week, month or total, filterable by ASIN, SKU, B2B/B2C and AFN/MFN. | Outside the Far East region: Inventory and Order Tracking or Product Listing | Hourly granularity covers ≤30 days. Other granularities cover ≤2 years. | [sales] [sales-model] [rmap] |
| **FBA Inventory v1** (`getInventorySummaries`) | Live FBA quantities (fulfillable, inbound, reserved, unfulfillable, researching) per SKU. | Amazon Fulfillment or Product Listing | `startDateTime` (changed-since) can be at most 18 months back. Rate 2 rps. | [fbainv] [fbainv-model] [rmap] |
| **Fulfillment Inbound v2024-03-20** ([inbound_plans](inbound_plans.md)) | Inbound plans, shipments, boxes, items, pallets, prep and compliance details, transport options. | Amazon Fulfillment | The v0 create/update operations were removed in Jan 2025. The v0 read operations (`getShipments`, `getShipmentItems`, labels, bill of lading) are still listed. | [inbound] [rmap] [deprec] |
| **Fulfillment Outbound v2026-07-04** | Multi-Channel Fulfillment orders, tracking, return reason codes, features. | Amazon Fulfillment | New version in 2026; v2020-07-01 is legacy | [outbound] [models] [relnotes] |
| **FBA Inbound Eligibility v1** | Whether an ASIN can be sent to FBA (inbound eligibility preview). | Amazon Fulfillment | — | [models] [rmap] |
| **Listings Items v2021-08-01** (`getListingsItem`, `searchListingsItems`) | Your listings: attributes, issues, offers, fulfilment availability, status. | Inventory and Order Tracking or Product Listing | Also reads and writes Automate Pricing rule assignments (with Product Type Definitions) | [listings] [rmap] [autoprice] |
| **Listings Restrictions v2021-08-01** | Whether you're restricted or gated from listing an ASIN, with the reason and approval links. | Product Listing | Useful for ungating checks | [restrictions] [rmap] |
| **Catalog Items v2022-04-01** | Catalogue data for any ASIN (attributes, images, ranks, relationships). | Product Listing | v0 operations are removed | [catalog] [deprec] |
| **Product Pricing v2022-05-01 + v0** | Competitive summary, Featured Offer Expected Price (up to 40 SKUs), top offers (`getItemOffers`), your pricing. | Pricing (v2022). Pricing or Product Listing (v0). | Live market data only; no price history | [pricing] [rmap] |
| **Product Fees v0** | Fee estimates per SKU or ASIN, or in batches of up to 20. | Per SKU or ASIN (`getMyFeesEstimateForSKU`, `getMyFeesEstimateForASIN`): any of Amazon Fulfillment, Buyer Communication, Buyer Solicitation, Finance and Accounting, Inventory and Order Tracking, Pricing, Product Listing, Selling Partner Insights, or the restricted Direct to Consumer Shipping, Professional Services, Tax Invoicing or Tax Remittance. Batch (`getMyFeesEstimates`): Pricing or Product Listing only | — | [fees] [rmap] |
| **Product Type Definitions v2020-09-01** | Listing attribute schemas. Also exposes your available Automate Pricing rules. | Inventory and Order Tracking or Product Listing | — | [ptd] [autoprice] |
| **Promotions 2025-12-01** (new) | Search and get your deals, coupons, price discounts and basket promotions, their item selections and validation issues. | Pricing or Product Listing | New in 2026 | [promos] [relnotes] |
| **Customer Feedback 2024-06-01** | Review topics and trends per ASIN or browse node. Return topics and trends per browse node only. | Brand Analytics or Selling Partner Insights | US, **UK**, FR, IT, DE, ES, JP. Weekly refresh, English only. | [cf] |
| **Replenishment v2022-11-07** | Subscribe & Save metrics (active subscriptions, revenue, out-of-stock losses, forecasts). | Brand Analytics or Inventory and Order Tracking | Trailing 2 years only | [replen] [replen-model] |
| **Notifications v1** | Push events to SQS or EventBridge (list in §4/§8). | Set by notification type | — | [notif] [rtypes] |
| **Feeds v2021-06-30** | Mostly a write API. Reads are limited to your own feed status and result documents. | Set by feed type | — | [feeds] |
| **Support API v2025-02-01** (new) | Your Seller Support cases: list, detail, status, full contact history (email, phone, chat), callbacks, attachments. | **Selling Partner Support (Restricted)** | All stores. Launched 2026. | [support] [roles] [relnotes] |
| **Messaging v1** | Only which message types you can send for an order (`getMessagingActionsForOrder`, `GetAttributes`). **It does not read buyer messages.** | Buyer Communication | Send-only API | [messaging] [rmap] |
| **Solicitations v1** | Whether you can request a review for an order. | Buyer Solicitation or Product Listing | Send-only | [solicit] [rmap] |
| **A+ Content 2020-11-01** | Your A+ documents and ASIN links. | Brand Analytics or Product Listing | A+ is a brand-owner feature (Brand Registry requirement **unverified** in the API docs) | [models] [rmap] |
| **Tokens v2021-03-01** | Issues Restricted Data Tokens. | Any of Amazon Fulfillment, Buyer Communication, Buyer Solicitation, Finance and Accounting, Inventory and Order Tracking, Pricing, Product Listing, Selling Partner Insights, or the restricted Direct to Consumer Shipping, Professional Services, Tax Invoicing or Tax Remittance (not Brand Analytics) | — | [tokens] [rdt] |
| **Merchant Fulfillment v0 / Shipping v2** | FBM label purchase, rates, tracking. | Merchant Fulfillment v0: Direct to Consumer Shipping (R). Shipping v2: Amazon Logistics or Direct to Consumer Shipping (R), except `purchaseShipment` (Amazon Logistics or Product Listing) and `getAccessPoints` (Direct to Consumer Shipping (R) only) | Only relevant if you ship FBM. For new builds, Merchant Fulfillment points to Shipping v2. | [mfn] [models] [rmap] |
| **Easy Ship v2022-03-23** | Easy Ship packages and slots. | Direct to Consumer Shipping (R) | **Not available in the UK** (MX, IN, TR, AU, SG, JP, NL, PL …) | [easyship] |
| **AWD v2024-05-09** | Amazon Warehousing & Distribution inventory and inbound shipments. | Amazon Warehousing and Distribution | **US store only** | [awd] [rmap] |
| **Supply Sources v2020-07-01** | Configured supply sources (BOPIS / store pickup). | Selling Partner Insights | Not relevant to FBA | [supply] [rmap] |
| **Services v1** | Home-services jobs. | Professional Services (R) | Not relevant | [services] |
| **Vehicles v2024-11-01** | Vehicle compatibility lists. | Product Listing (EU region) | Auto parts only | [vehicles] |
| **Invoices v2024-06-19 (seller)** | FBA invoices | Tax Invoicing (R) | **Brazil only** | [models] [invoices] |
| Shipment Invoicing v0, Delivery by Amazon | Brazil FBA invoicing | Tax Invoicing (R) | **Brazil only** | [models] |
| External Fulfillment (Inventory / Returns / Shipping v2024-09-11, Location v2026-07-30) | For sellers fulfilling Amazon orders from their own warehouses | Direct-to-Consumer Shipping (R) | Not relevant to FBA | [models] [rmap] [llms] |
| **Tracking 2026-01-30** (new) | Tracking milestones for external-channel shipments | Grantless | **US only** | [tracking] |
| App Integrations v2024-04-01 | Sends app notifications into Seller Central (writes, not reads) | Notifications in Seller Central | — | [appint] |
| Application Management v2023-11-30 | Rotates your app's client secret | Grantless | — | [appmgmt] |
| Uploads v2020-11-01 | Uploads files (write) | Product Listing | — | [rmap] |
| FBA Small and Light v1 | — | — | **Removed Sept 2023 (NA/EU)** | [deprec] |
| Vendor-only APIs (Vendor Orders, Shipments, Invoices, Direct Fulfillment …, Finance Remittance 2026-03-17, Invoices v2026-06-25) | Not applicable to Seller Central | — | Vendors only | [models] |

**There is no "Seller Performance" or "Account Health" API section.** Account health comes through Reports, Data Kiosk and Notifications (§4) **[models]**.

---

## 2. Reports API: every seller report type, grouped as Amazon groups them

Notes that apply to every report:

- **Retention:** report documents are kept for at most **90 days**, and `getReports.createdSince` can't look further back **[reports-model]**. A report type with no stated retention is kept 90 days **[rt]**.
- **Duplicate requests:** some reports can be requested only once per period. Later requests are **CANCELLED**. A report with no data is also CANCELLED **[reports-faq]**.
- **Seller Central has more reports than the API.** Amazon says some Seller Central reports aren't available through the API **[reports-faq]**.
- **Inventory/listing reports are cached for 1–6 hours.** A repeat request inside that window returns the cached copy **[rt-inv]**.

Column key: **Req** = requested on demand, **Sch** = can be scheduled, **Auto** = Amazon-generated. "(R)" = restricted role, and RDT means a Restricted Data Token is needed to download.

### 2.1 Analytics (Seller Retail Analytics + Brand Analytics) [rt-analytics] [rtypes]

| reportType | Content | How | Role / requirement | History / range |
|---|---|---|---|---|
| [`GET_SALES_AND_TRAFFIC_REPORT`](sales_and_traffic.md) | Business Reports: sales and traffic by date and by parent ASIN, child ASIN or SKU, including B2B. JSON. | Req + Sch | **Brand Analytics** role. Brand Registry isn't stated as needed. | Start date at most **2 years** back. Granularity DAY/WEEK/MONTH. 3 requests per 5 minutes. |
| `GET_BRAND_ANALYTICS_SEARCH_TERMS_REPORT` | Top clicked ASINs per search term | Req | Brand Analytics + **Brand Registry** | DAY/WEEK/MONTH/QUARTER; can't span periods |
| `GET_BRAND_ANALYTICS_SEARCH_QUERY_PERFORMANCE_REPORT` | Query-level funnel for your ASINs | Req | Brand Analytics + Brand Registry | WEEK/MONTH/QUARTER; MONTH max range 17 months |
| `GET_BRAND_ANALYTICS_SEARCH_CATALOG_PERFORMANCE_REPORT` | ASIN funnel (impressions to purchases) | Req | Brand Analytics + Brand Registry | WEEK/MONTH/QUARTER |
| `GET_BRAND_ANALYTICS_MARKET_BASKET_REPORT` | Bought-together products | Req | Brand Analytics + Brand Registry | DAY to QUARTER |
| `GET_BRAND_ANALYTICS_REPEAT_PURCHASE_REPORT` | Repeat purchase metrics | Req | Brand Analytics + Brand Registry | WEEK to QUARTER |
| `GET_VENDOR_*` (8 types) | Vendor only | Brand Analytics | — | — |

### 2.2 Order reports (restricted, contain PII) [rt-order] [rtypes]

Retained 30 days. Covers only orders **less than 2 years old**. All of these need an RDT to download.

| reportType | Content | How | Role | Range |
|---|---|---|---|---|
| `GET_FLAT_FILE_ACTIONABLE_ORDER_DATA_SHIPPING` | Unshipped FBM orders | Req/Sch | Direct to Consumer Shipping (R) | 38-day lookback |
| `GET_ORDER_REPORT_DATA_INVOICING` | XML, EU tax invoicing | **Sch only** | Tax Invoicing (R), EU | past 38 days, then new orders |
| `GET_ORDER_REPORT_DATA_TAX` | XML, NA tax | Sch only | Tax Remittance (R), NA | 38 days |
| `GET_ORDER_REPORT_DATA_SHIPPING` | XML, FBM shipping | Sch only | Direct to Consumer Shipping (R) | 38 days |
| `GET_FLAT_FILE_ORDER_REPORT_DATA_INVOICING` | Flat file, EU invoicing | Req/Sch | Tax Invoicing (R), EU | previous 60 days |
| `GET_FLAT_FILE_ORDER_REPORT_DATA_SHIPPING` | Flat file, shipping | Req/Sch | Direct to Consumer Shipping (R) | 60 days |
| `GET_FLAT_FILE_ORDER_REPORT_DATA_TAX` | Flat file, NA tax | Req/Sch | Tax Remittance (R) | 60 days |

### 2.3 Order tracking reports (no PII, all channels incl. FBA) [rt-order] [rt-fba]

| reportType | Content | How | Role (any one) | Range |
|---|---|---|---|---|
| `GET_FLAT_FILE_ALL_ORDERS_DATA_BY_LAST_UPDATE_GENERAL` | Orders updated in the period | Req only | Amazon Fulfillment, Inventory and Order Tracking, Pricing, Product Listing, DTC Shipping (R) | **Max 30-day range. "Start date can be any date"**, but the Orders API caps at 2 years. Actual depth beyond 2 years is **unverified**. |
| [`GET_FLAT_FILE_ALL_ORDERS_DATA_BY_ORDER_DATE_GENERAL`](orders.md) | Orders placed in the period | Req | Amazon Fulfillment, Inventory and Order Tracking, Pricing, DTC Shipping (R) (not Product Listing) | 30-day range |
| `GET_FLAT_FILE_ARCHIVED_ORDERS_DATA_BY_ORDER_DATE` | Archived orders | Req | Amazon Fulfillment, Inventory and Order Tracking, Pricing, DTC Shipping (R) (not Product Listing) | 30-day range |
| `GET_XML_ALL_ORDERS_DATA_BY_LAST_UPDATE_GENERAL` / `…_BY_ORDER_DATE_GENERAL` | XML versions | Req | Amazon Fulfillment, Inventory and Order Tracking, Pricing, DTC Shipping (R) (not Product Listing) | 30-day range |

### 2.4 Pending order reports [rt-order]

`GET_FLAT_FILE_PENDING_ORDERS_DATA` (Req/Sch, JP), `GET_PENDING_ORDERS_DATA` (Sch, JP), both Inventory and Order Tracking, and `GET_CONVERGED_FLAT_FILE_PENDING_ORDERS_DATA` (Req/Sch, Product Listing). All are pending FBM orders with a 38-day lookback.

### 2.5 Returns reports: FBM (seller-fulfilled) returns only [rt-ret]

All six are Req/Sch and allow up to **60 days of data per report**. Roles: Inventory and Order Tracking or DTC Shipping (R). The two Prime variants and `GET_FLAT_FILE_MFN_SKU_RETURN_ATTRIBUTES_REPORT` also accept Pricing; `GET_XML_MFN_SKU_RETURN_ATTRIBUTES_REPORT` doesn't. **They exclude FBA returns**; use [`GET_FBA_FULFILLMENT_CUSTOMER_RETURNS_DATA`](fba_returns.md) for those.

`GET_XML_RETURNS_DATA_BY_RETURN_DATE`, [`GET_FLAT_FILE_RETURNS_DATA_BY_RETURN_DATE`](fbm_returns.md), `GET_XML_MFN_PRIME_RETURNS_REPORT`, `GET_CSV_MFN_PRIME_RETURNS_REPORT`, `GET_XML_MFN_SKU_RETURN_ATTRIBUTES_REPORT`, `GET_FLAT_FILE_MFN_SKU_RETURN_ATTRIBUTES_REPORT`.

### 2.6 Performance [rt-perf] [rtypes]

| reportType | Content | How | Role | Range |
|---|---|---|---|---|
| [`GET_V2_SELLER_PERFORMANCE_REPORT`](account_health.md) | **Account Health dashboard**: account status per store, ODR (AFN/MFN), late shipment, pre-fulfilment cancel, valid tracking, on-time delivery (incl. unit level), invoice defect rate, **Account Health Rating** (status + score), policy violations (listing, restricted products, suspected IP, food/product safety, review policy, other), customer complaints (authenticity, condition, safety, received IP), document requests and warnings. JSON. | Req | Selling Partner Insights | Current snapshot. No date range is documented. "Warning states" is deprecated and always empty. |
| [`GET_V1_SELLER_PERFORMANCE_REPORT`](account_health_v1.md) | XML customer metrics: ODR, cancellation, late shipment, policy violations, contact response time, return dissatisfaction etc. | Req | Selling Partner Insights | Not stated |
| [`GET_SELLER_FEEDBACK_DATA`](seller_feedback.md) | Negative and neutral (1–3 star) feedback with comments, your response, order ID. Rater email is deprecated. | Req | Selling Partner Insights or Amazon Fulfillment | Not stated |
| `GET_PROMOTION_PERFORMANCE_REPORT` | Best Deal / Lightning Deal performance (glance views, units, revenue) | Req | Selling Partner Insights or Brand Analytics (Amazon's report page: Selling Partner Insights for sellers, Brand Analytics for vendors) | Promotion start up to **2 years** back |
| `GET_COUPON_PERFORMANCE_REPORT` | Coupon clips, redemptions, budget, sales | Req | **Brand Analytics** | Up to **2 years** back |

### 2.7 Settlement / payments [rt-settle] [rt-pay] [deprec] [auto-reports]

| reportType | Content | How | Role | Range |
|---|---|---|---|---|
| [`GET_V2_SETTLEMENT_REPORT_DATA_FLAT_FILE_V2`](settlements.md) | Settlement report (current format) | **Auto only.** It can't be requested or scheduled; you find it with `getReports`. | Finance and Accounting | Only what's in the 90-day report retention **[reports-model]** |
| `GET_V2_SETTLEMENT_REPORT_DATA_FLAT_FILE` / `…_XML` | Old formats | Auto | Finance and Accounting (flat file also Amazon Fulfillment) | **Removed 11 Nov 2026** |
| `GET_DATE_RANGE_FINANCIAL_HOLDS_DATA` | Deferred Transaction Report | **Must first be requested in Seller Central** (Payments Reports Repository), then fetched by API | Finance and Accounting | Up to **10 years** in one report |

### 2.8 FBA reports [rt-fba] [rtypes]

Generation limits: "near real-time" reports regenerate at most every 30 minutes, and "daily" reports at most every 4 hours.

| Sub-group | reportType | Content | How | Role | History / limits |
|---|---|---|---|---|---|
| Sales | [`GET_AMAZON_FULFILLED_SHIPMENTS_DATA_GENERAL`](fba_shipments.md) | Shipment-level FBA order data (price, carrier, tracking; buyer name, email and phone fields) | Req | Amazon Fulfillment, Inventory and Order Tracking, Pricing | **≤1 month per report. Max 12 requests per 8 hours** (Seller Central requests count too) |
| Sales | `GET_AMAZON_FULFILLED_SHIPMENTS_DATA_INVOICING` | EU invoicing variant | Req | **Tax Invoicing (R)** + RDT | ≤1 month |
| Sales | `GET_AMAZON_FULFILLED_SHIPMENTS_DATA_TAX` | NA tax variant | Req | Tax Remittance (R) + RDT | ≤1 month |
| Sales | [`GET_FBA_FULFILLMENT_CUSTOMER_SHIPMENT_SALES_DATA`](fba_shipment_sales.md) | Item-level shipped sales | Req | Amazon Fulfillment | Not stated |
| Sales | `GET_FBA_FULFILLMENT_CUSTOMER_SHIPMENT_PROMOTION_DATA` | Promotions on FBA orders | Req | Amazon Fulfillment, Pricing | **NA only** |
| Sales | `GET_FBA_FULFILLMENT_CUSTOMER_TAXES_DATA` | Legacy US tax | Req | Amazon Fulfillment or Tax Remittance (R) | Data only up to Feb 2013 (effectively dead) |
| Sales | `GET_REMOTE_FULFILLMENT_ELIGIBILITY` | NARF eligibility | Req | Amazon Fulfillment or Inventory and Order Tracking | US |
| Inventory | `GET_AFN_INVENTORY_DATA` | Fulfillable quantity by SKU and condition | Req | Amazon Fulfillment | Live |
| Inventory | [`GET_AFN_INVENTORY_DATA_BY_COUNTRY`](inventory_by_country.md) | Multi-country inventory | Req | Amazon Fulfillment, Pricing | EU only |
| Inventory | [`GET_LEDGER_SUMMARY_VIEW_DATA`](ledger_summary.md) | Inventory "bank statement" (opening balance, receipts, shipments, returns, adjustments, removals, closing balance) | Req | Amazon Fulfillment | Range must align to the period (DAILY/WEEKLY/MONTHLY). History not stated. |
| Inventory | [`GET_LEDGER_DETAIL_VIEW_DATA`](ledger_detail.md) | Every inventory movement (sales, returns, removals, damage, lost, found…) | Req | Amazon Fulfillment | **18 months** |
| Inventory | [`GET_RESERVED_INVENTORY_DATA`](reserved_inventory.md) | Reserved units by reason | Req | Amazon Fulfillment, Pricing | Live |
| Inventory | `GET_FBA_MYI_UNSUPPRESSED_INVENTORY_DATA` / [`GET_FBA_MYI_ALL_INVENTORY_DATA`](inventory_all.md) | Manage Inventory, without / with archived | Req | Amazon Fulfillment, Pricing | Live |
| Inventory | [`GET_RESTOCK_INVENTORY_RECOMMENDATIONS_REPORT`](restock.md) | Restock recommendations | Req | Amazon Fulfillment, Pricing | Live; not available in IN |
| Inventory | [`GET_FBA_FULFILLMENT_INBOUND_NONCOMPLIANCE_DATA`](inbound_noncompliance.md) | Inbound problems | Req | Amazon Fulfillment, Pricing | Daily |
| Inventory | [`GET_STRANDED_INVENTORY_UI_DATA`](stranded.md) / `GET_STRANDED_INVENTORY_LOADER_DATA` | Stranded inventory | Req | Amazon Fulfillment, Pricing | Live |
| Inventory | [`GET_FBA_INVENTORY_PLANNING_DATA`](inventory_planning.md) | **Inventory Health / age**: 7/30/60/90-day sales, aged and excess units, estimated fees, recommendations. Replaces the old aged-inventory reports. | Req | Amazon Fulfillment | Snapshot. Not in NL/PL/SE/BE. |
| Inventory | [`GET_FBA_STORAGE_FEE_CHARGES_DATA`](storage_fees.md) | Estimated monthly storage fee per ASIN | **Req + Sch** | Amazon Fulfillment, Pricing | Not stated |
| Inventory | `GET_FBA_OVERAGE_FEE_CHARGES_DATA` | Storage overage fees | Req | Amazon Fulfillment | Not stated |
| Payments | [`GET_FBA_ESTIMATED_FBA_FEES_TXT_DATA`](fba_fee_preview.md) | FBA fee preview | Req, **1 per day** | Amazon Fulfillment, Pricing | `dataStartTime` ≥72 hours ago |
| Payments | [`GET_FBA_REIMBURSEMENTS_DATA`](reimbursements.md) | Itemised reimbursements with reasons | Req | Amazon Fulfillment, Pricing | Daily. **History not stated.** |
| Payments | [`GET_FBA_FULFILLMENT_LONGTERM_STORAGE_FEE_CHARGES_DATA`](long_term_storage_fees.md) | Long-term storage charges | Req | Amazon Fulfillment, Pricing | Range should be one month |
| Concessions | `GET_FBA_FULFILLMENT_CUSTOMER_RETURNS_DATA` | FBA customer returns with reason and disposition | Req | Amazon Fulfillment, Pricing | Daily; history not stated |
| Concessions | `GET_FBA_FULFILLMENT_CUSTOMER_SHIPMENT_REPLACEMENT_DATA` | Replacements | Req | Amazon Fulfillment | **NA and IN only** |
| Removals | [`GET_FBA_RECOMMENDED_REMOVAL_DATA`](recommended_removals.md) | Recommended removals | Req | Amazon Fulfillment, Pricing | Daily |
| Removals | [`GET_FBA_FULFILLMENT_REMOVAL_ORDER_DETAIL_DATA`](removal_orders.md) | Removal orders | Req | Amazon Fulfillment, Pricing | Live |
| Removals | [`GET_FBA_FULFILLMENT_REMOVAL_SHIPMENT_DETAIL_DATA`](removal_shipments.md) | Removal shipments and tracking | Req | Amazon Fulfillment, Pricing | Live |
| Subscribe & Save | `GET_FBA_SNS_FORECAST_DATA`, `GET_FBA_SNS_PERFORMANCE_DATA` | — | — | — | **Removed 11 Dec 2025.** Use the Replenishment API. **[deprec]** |
| Small & Light | — | — | — | — | No current report on the FBA page. The S&L API was removed in 2023 **[deprec]**. |

### 2.9 Inventory / listings reports [rt-inv] [rtypes]

Roles for most of these: any one of Inventory and Order Tracking, Pricing, Product Listing or DTC Shipping (R). Exceptions: `GET_MERCHANT_LISTINGS_INACTIVE_DATA` doesn't take DTC Shipping; `GET_PAN_EU_OFFER_STATUS` also takes Amazon Fulfillment; `GET_MERCHANTS_LISTINGS_FYP_REPORT` is Product Listing only and `GET_REFERRAL_FEE_PREVIEW_REPORT` Finance and Accounting only; Amazon names no role for `GET_MFN_PANEU_OFFER_STATUS`. All are Req only, except [`GET_MERCHANTS_LISTINGS_FYP_REPORT`](suppressed_listings.md), which is Req/Sch. All are current-state snapshots.

| reportType | Content |
|---|---|
| `GET_FLAT_FILE_OPEN_LISTINGS_DATA` | SKU, ASIN, price, quantity, business/quantity pricing |
| [`GET_MERCHANT_LISTINGS_ALL_DATA`](listings_all.md) | All listings incl. status and fulfilment channel |
| `GET_MERCHANT_LISTINGS_DATA` | Active listings |
| `GET_MERCHANT_LISTINGS_INACTIVE_DATA` | Inactive listings |
| `GET_MERCHANT_LISTINGS_DATA_BACK_COMPAT` | Open listings (legacy format) |
| `GET_MERCHANT_LISTINGS_DATA_LITE` / `…_LITER` | SKU, price and quantity / SKU and quantity |
| `GET_MERCHANT_CANCELLED_LISTINGS_DATA` | Cancelled listings |
| `GET_MERCHANTS_LISTINGS_FYP_REPORT` | **Suppressed listings** with reason and fix (Product Listing role) |
| [`GET_PAN_EU_OFFER_STATUS`](pan_eu_status.md) / `GET_MFN_PANEU_OFFER_STATUS` | Pan-EU eligibility (UK, DE, FR, IT, ES) |
| `GET_REFERRAL_FEE_PREVIEW_REPORT` | Estimated referral fee per SKU (Finance and Accounting; once per 24 hours) |

### 2.10 Invoice data [rt-invoice]

`GET_FLAT_FILE_VAT_INVOICE_DATA_REPORT` and `GET_XML_VAT_INVOICE_DATA_REPORT` hold the data needed to raise VAT invoices, for sellers enrolled in VCS who upload their own invoices. They cover UK, IE, DE, FR, IT, ES, NL, BE, SE and PL, and need **Tax Invoicing (R)**. With `all=true` the max range is 30 days. With `pendingInvoices` the window is orders from the previous 90 days.

### 2.11 Tax reports [rt-tax]

| reportType | Stores | How | Role | Notes |
|---|---|---|---|---|
| `SC_VAT_TAX_REPORT` | **UK**, IE, DE, FR, IT, ES, NL, BE, SE, PL | Req | **Tax Invoicing (R)** + RDT | Amazon VAT Calculation Report: shipments, returns and refunds with VAT detail |
| `GET_VAT_TRANSACTION_DATA` | **UK** + EU | Req | **Tax Invoicing (R)** + RDT | Amazon VAT Transactions Report: sales, returns, refunds, cross-border movements and FC transfers. **Available from the 3rd of each month.** Returns all stores regardless of `marketplaceId`. |
| `GET_FLAT_FILE_SALES_TAX_DATA` | US | **Can't be requested.** Generate it in the Seller Central Tax Document Library, then list it. | Tax Remittance (R) | — |
| `GST_MTR_*`, `GET_GST_*` | India | — | — | Not relevant |

### 2.12 Regulatory compliance [rt-reg]

| reportType | Content | How | Role | Notes |
|---|---|---|---|---|
| `MARKETPLACE_ASIN_PAGE_VIEW_METRICS` | ASIN page (glance) views | Req/Sch | Selling Partner Insights | **UK**, IE, EU-8. Data up to the last 7 days. No data where you sold <40 units in a store/category. |
| `END_USER_DATA_REPORT` | Customer data for customers who opted in to share it | Req/Sch | Buyer Communication | EU stores only (not UK) |
| `FBA_BULK_INVOICE` | Tar.gz of PDF invoices for a date range | Req | Tax Invoicing (R) | — |
| `GET_EPR_MONTHLY_REPORTS` / `…_QUARTERLY_…` / `…_ANNUAL_…` | Extended Producer Responsibility data (units sold, packaging weights) | Req | Product Listing | — |

### 2.13 Amazon Business, B2B, Easy Ship, Browse Tree

- `FEE_DISCOUNTS_REPORT`: referral-fee discounts for Amazon Business sellers. Req/Sch, Product Listing role, once per 24 hours **[rt-ab]**.
- `GET_B2B_PRODUCT_OPPORTUNITIES_RECOMMENDED_FOR_YOU` and `GET_B2B_PRODUCT_OPPORTUNITIES_NOT_YET_ON_AMAZON`: up to 10,000 sourcing recommendations, refreshed weekly. Includes **UK**. Product Listing role **[rt-b2b]**.
- `GET_EASYSHIP_DOCUMENTS`: DTC Shipping (R) or Tax Invoicing (R). `GET_EASYSHIP_PICKEDUP`, `GET_EASYSHIP_WAITING_FOR_PICKUP`: DTC Shipping (R). Easy Ship isn't available in the UK **[rt-es] [easyship]**.
- `GET_XML_BROWSE_TREE_DATA`: the category tree. Req/Sch. Roles: Inventory and Order Tracking, Pricing, Product Listing or DTC Shipping (R) **[rt-bt]**.

### 2.14 Report types in Amazon's role map but not on any report-type page (availability unverified) [rtypes]

`GET_FBA_FULFILLMENT_INVENTORY_EXPIRATION_DATE_DATA`, `GET_MERCHANT_LISTINGS_DEFECT_DATA`, `GET_FLAT_FILE_GEO_OPPORTUNITIES`, `GET_OUT_OF_STOCK_MFN_INFORMATION`, `GET_CONVERGED_FLAT_FILE_SOLD_LISTINGS_DATA`, `GET_FLAT_FILE_ORDERS_RECONCILIATION_DATA_*`, `GET_ORDER_REPORT_DATA_ALL`, `GET_XML_ALL_ORDERS_DATA_BY_LAST_UPDATE_ALL`, `SC_VAT_TAX_REPORT_INVOICE`, `GET_FLAT_FILE_OFFER_POINTS_PREFERENCE_DATA`, `RFQD_BULK_DOWNLOAD`, `WARRANTY_BASE_ITEM_INFO`.

**Promotions, coupons, Vine, Subscribe & Save:**

- Promotions and coupons: see §2.6 and the Promotions API.
- Subscribe & Save: the Replenishment API.
- **Vine:** no report or API found anywhere in the SP-API docs index (**unverified** absence) **[llms]**.

---

## 3. Data Kiosk datasets [dk] [dk-schemas]

How Data Kiosk works:

- **Queries and results:** you send GraphQL queries and get JSONL results.
- **One query at a time per dataset:** only one query can be running per query / selling partner / application, and a new query fails while one is pending **[dk]**.
- **No scheduling or backlog:** Data Kiosk doesn't support query scheduling or query backlogging. Amazon's guide shows how to build your own scheduler **[dk] [dk-sched]**.
- **Rate limits:** the same as the Reports API **[dk-wf]**.
- **Role:** calling the API needs **Brand Analytics**, and roles are also checked per field **[dk]**. The Economics dataset also needs **Selling Partner Insights** **[dk-roles]**.
- **Result retention:** most fields carry `@resultRetention(duration: "P30D")`, so result documents are kept 30 days **[dk-econ] [dk-st]**.

| Dataset | What's in it | History | Notes |
|---|---|---|---|
| **Seller Analytics** `analytics_sellerAnalytics_2025_03_31` (added Sept 2026; pack pages [seller_analytics](seller_analytics.md) per store with account totals, [seller_analytics_asin](seller_analytics_asin.md) per ASIN) | 100+ metrics in one view (`sellerAnalyticsView`): **sales** (ordered sales, units, ASP, GMS); **traffic** (sessions, page views, featured-offer %, conversion); **promotion/deals**; **accountHealth** (ODR, negative feedback, A-to-z, chargeback, late shipment, pre-fulfilment cancel, valid tracking, on-time delivery rates and counts, plus `accountHealthRatingScore`); 120-day product-defect snapshots; **search** (keyword-level impressions, clicks, cart adds, purchases); **reviews** (low-rating count, critical review rate); **defects** (return-defect topics with recommendations); **returns** (returned units, refund amount, return rate). Group by ASIN, MSKU, brand, parent ASIN, fulfilment channel, product group, store. Multi-store within one region. Week-over-week, month-over-month and year-over-year benchmarks. | `startDate` ≤ **2 years** ago. Snapshot metrics are limited (daily = single date; 8 weeks; 24 months). Returns lag about 45 days. | Seller accounts only. The "lending" metric group and seller attributes need a separate "sales and lending" access level. The B2B metrics are declared but **not yet served**. | [dk-sa] [relnotes] |
| **Sales and Traffic** `analytics_salesAndTraffic_2024_04_24` | Business Report metrics by ASIN, by date, and trends: ordered and shipped sales, units, refunds, refund rate, sessions, page views, **Buy Box %**, unit session %, feedback received, negative feedback rate, claims granted and amount, average offer count, plus B2B variants. | Start date ≤ **2 years** ago | The older `…_2023_11_15` version was **removed 27 Mar 2026** | [dk-st] [deprec] [relnotes] |
| **Seller Economics** [`analytics_economics_2024_03_15`](seller_economics.md) | Per MSKU/ASIN per day, week or month: sales (ordered, net, refunded, units), **every Amazon fee by type** (with components, e.g. FBA fulfilment, referral, storage, deal, coupon, removal, disposal, fuel), **ad spend by ad type**, per-unit costs (COGS, MFN and FBA costs, shipping to Amazon, misc. costs, as entered in SKU Central or the Revenue Calculator), net proceeds. Also `economicsPreview` (fees up to 120 days ahead) and `economicsSimulation` (what-if, up to 100 products). | Start date ≤ **2 years** ago. Refreshed daily. | Effectively a per-SKU profit and loss, the closest thing to an "all fees" ledger | [dk-econ] [relnotes] |
| Vendor Analytics `analytics_vendorAnalytics_2024_09_30` | Vendor only | — | Not applicable | [dk-schemas] |

---

## 4. Account health / performance data via the API: what exists and what doesn't

| Source | What you get | Role | Source |
|---|---|---|---|
| [`GET_V2_SELLER_PERFORMANCE_REPORT`](account_health.md) | The Account Health dashboard as JSON (full list in §2.6), incl. **Account Health Rating** and all policy-violation and complaint categories | Selling Partner Insights | [rt-perf] |
| [`GET_V1_SELLER_PERFORMANCE_REPORT`](account_health_v1.md) | Older XML customer metrics, incl. contact response time | Selling Partner Insights | [rt-perf] |
| [`GET_SELLER_FEEDBACK_DATA`](seller_feedback.md) | Negative and neutral feedback | Selling Partner Insights / Amazon Fulfillment | [rt-perf] |
| Data Kiosk **Seller Analytics** `accountHealth` group | Account-health rates and counts **by period, up to 2 years back**, plus `accountHealthRatingScore` and 120-day product-defect snapshots | Brand Analytics (to call Data Kiosk) | [dk-sa] |
| Data Kiosk Sales & Traffic | Feedback received, negative feedback rate, claims granted and amount, by date | Brand Analytics | [dk-st] |
| Notification `ACCOUNT_STATUS_CHANGED` | Push alert when account status moves between `NORMAL`, `AT_RISK` and `DEACTIVATED` | Selling Partner Insights | [notif] [rtypes] |
| Notifications `LISTINGS_ITEM_ISSUES_CHANGE`, `LISTINGS_ITEM_STATUS_CHANGE`, `PRICING_HEALTH` | Listing problems, buyability changes, offer losing Featured Offer eligibility because of price | Product Listing / Pricing | [notif] [rtypes] |
| Sellers API [`getMarketplaceParticipations`](store_list.md) | `hasSuspendedListings` per store | Selling Partner Insights / Product Listing | [sellers-model] |
| Listings Restrictions API | Whether you're gated per ASIN | Product Listing | [restrictions] |
| Support API (new) | Your Seller Support cases and full contact history | Selling Partner Support (Restricted) | [support] |

**Not available (as documented):**

- There's **no dedicated Account Health API** in Amazon's API list **[models]**.
- The V2 report's warning-states section is deprecated and always empty **[rt-perf]**.
- No date-range or history parameter is documented for the V1/V2 performance reports. Treat them as current-state, and keep your own history (or use Data Kiosk) **[rt-perf]**.
- No API found for reading the **Performance Notifications inbox**, appealing, or submitting a Plan of Action. The Support API reads cases only, and the listed operations are all reads **[rmap]** (the absence of an appeals API is **unverified** beyond the API list).

---

## 5. Restricted data / PII

**Restricted roles:**

- Direct-to-Consumer Shipping (R)
- Tax Invoicing (R)
- Tax Remittance (R)
- Professional Services (R)
- Selling Partner Support (R)

For these roles "you must provide additional information about your data use and security controls" **[roles]**.

**When an RDT is needed:**

- **Restricted report types** need an RDT to call `getReportDocument`. The RDT must name the specific report-document path **[rdt] [data-access]**.
- **Orders v2026-01-01 needs no RDT**; the roles alone gate PII **[orders-pii] [orders-mig]**.
- Orders v0 still uses RDTs (getOrderBuyerInfo and the like) until it's removed in Mar 2027 **[deprec]**.

**What PII the Orders API gives for UK/EU (non-US/JP/SG) orders [orders-pii]:**

| Field | No restricted role | Tax Invoicing / Tax Remittance (R) | DTC Shipping (R) |
|---|---|---|---|
| Buyer email | No | No | **FBM only** |
| Buyer name | No | **FBA + FBM** | FBM only |
| Delivery name and address lines | No | **FBA + FBM** | FBM only |
| Phone | No | No | FBM only (suppressed after the latest delivery date) |
| Postcode | **Yes** | Yes | Yes |
| Buyer tax registrations | No | FBA + FBM | No |
| Gift message / customisation | No | No | FBM only |

This table covers the Orders API only. The Amazon Fulfilled Shipments report ([fba_shipments](fba_shipments.md)) has a `buyer-email` column that can hold the buyer's email on UK orders, with none of the restricted roles.

**Restricted reports that matter to a UK FBA seller:**

- `SC_VAT_TAX_REPORT`, `GET_VAT_TRANSACTION_DATA`, `GET_FLAT_FILE_VAT_INVOICE_DATA_REPORT`, `GET_AMAZON_FULFILLED_SHIPMENTS_DATA_INVOICING` and `FBA_BULK_INVOICE` all need **Tax Invoicing (R)** **[rt-tax] [rt-invoice] [rt-fba] [rt-reg]**.
- The three shipping order reports in §2.2 (`GET_FLAT_FILE_ACTIONABLE_ORDER_DATA_SHIPPING`, `GET_ORDER_REPORT_DATA_SHIPPING`, `GET_FLAT_FILE_ORDER_REPORT_DATA_SHIPPING`) need DTC Shipping (R); the two invoicing ones need Tax Invoicing (R) and the two tax ones Tax Remittance (R). The FBM returns reports accept DTC Shipping (R) but don't need it: Inventory and Order Tracking is enough **[rt-order] [rt-ret]**.

**Can a private seller app get restricted roles, and how hard is approval?**

- **What the docs say:**
  - Private developers pick roles in the same Developer Profile, and restricted roles are among those listed **[reg-priv] [roles]**.
  - Amazon reviews every role request, "with additional scrutiny for restricted roles" **[rmap]**.
  - Amazon says it "cannot provide detailed information about how we determine Selling Partner API eligibility, but we prioritize data security" **[reg-priv]**.
  - You must answer Amazon's questions within 5 days or the case closes **[reg-priv]**.
- **Policy constraints you'd sign up to:**
  - The Data Protection Policy says PII is granted "for selected tax and merchant fulfilled shipping purposes, on a must-have basis".
  - PII may be kept **no longer than 30 days after order delivery** unless the law requires longer.
  - PII must be encrypted at rest (AES-128 or RSA-2048 or stronger).
  - You must report security incidents within 24 hours.
  - Amazon may audit you **[dpp]**.
  - The Acceptable Use Policy says customer PII may not be used for anything except merchant-fulfilled shipping, with tax and legal exceptions **[aup]**.
- **Practical reading (inference, not stated by Amazon):**
  - Tax Invoicing (R) has a clear purpose for a UK VAT-registered seller who needs the VAT reports, so it's the most justifiable restricted role.
  - DTC Shipping (R) is hard to justify for a pure-FBA seller.
  - Actual approval odds and timelines aren't published (**unverified**).
- **Open-banking roles (AISP/PISP)** need a licensed payment-provider orgID (e.g. FCA). They're not available to an ordinary seller **[tpp]**.

---

## 6. Private (self-authorised) developer app for your own account

**Steps [reg] [reg-priv] [selfauth] [data-access]:**

1. You must have a **Professional** selling account (Individual accounts aren't eligible), and you must be the **primary account user** **[reg]**.
2. Go to the Solution Provider Portal (solutionproviderportal.amazon.com), or in Classic Seller Central choose **Apps and Services → Develop Apps** (the new menu has no entry for it, checked 28 Oct 2026); it asks you to sign in again with your Seller Central login. Complete the Developer Profile and choose *"Private Developer: I build application(s) that integrate my own company with Amazon Services APIs"*. Fill in roles, use cases and security controls, then accept the Solution Provider Portal Agreement, AUP and DPP **[reg-priv]**.
3. Amazon reviews the profile and opens a case. Reply within 5 days **[reg-priv]**.
4. Register an app (it can stay in draft, because private apps are never published). Click **Authorize app** to get a **refresh token**, then exchange it at `https://api.amazon.com/auth/o2/token` for 1-hour access tokens **[selfauth]**.
5. You only register once, in any region **[data-access]**. A private app allows **10 self-authorisations** and no OAuth **[authlimits]**. Merged EU/NA accounts get an **Authorize app** button for each region **[selfauth]**.
6. **Credential housekeeping:**
   - LWA client secrets must be rotated every 180 days: "You must rotate your LWA credentials (client secrets) for all applications every 180 days." Amazon sends a notification 90 days before the deadline, and the old secret expires seven days after the new one is made **[lwarot]**. You rotate it in the Solution Provider Portal or with the Application Management API **[lwarot] [credrot] [appmgmt]**.
   - The AUP says "Access keys that do not make a successful call in 90 days will be deleted" **[aup]**.

**Roles to request for a read-everything private app** (built from §1–§4):

- **Non-restricted:**
  - Selling Partner Insights: performance reports, account status, page views, the Data Kiosk Economics dataset (with Brand Analytics).
  - Finance and Accounting: finances, settlements, payouts, Sellers [`getAccount`](account.md), referral fee preview.
  - Amazon Fulfillment: FBA reports, inventory, inbound, MCF.
  - Inventory and Order Tracking: order reports, Sales API, listings.
  - Product Listing: catalog, listings, restrictions, EPR, B2B opportunities.
  - Pricing: pricing, offers, Promotions API.
  - **Brand Analytics:** Sales & Traffic report, Data Kiosk, coupon report, Customer Feedback.
- **Optional restricted:** Tax Invoicing (UK VAT reports and buyer name/address on FBA orders), and Selling Partner Support (support cases).
- **Unverified:** whether Amazon grants the Brand Analytics role to a reseller who isn't the brand owner. The docs say the role is "available to sellers" and that Brand Analytics *reports* need Brand Registry, but they don't say the role itself needs it **[roles] [rt-analytics]**.

**SP-API fees (2025–2026):**

| Date | Event | Source |
|---|---|---|
| Nov 2025 | Amazon announced a **$1,400/yr subscription** for third-party developers from 31 Jan 2026, plus a **monthly GET-call usage fee** from 30 Apr 2026. Tiers: Basic $0 with 2.5M GET calls; Pro and Plus higher; reported overage $0.40 per 1,000 GET calls. | [gh5025] [novadata] |
| Same announcement | Reported scope: "Sellers and vendors using SP-API directly for only their own businesses will not face additional SP-API fees." This is secondary reporting. Amazon's original announcement page (`developer.amazonservices.com/spp-announcement`) now returns 404, so it couldn't be checked against the primary source. | [ppcland] |
| 9 Mar 2026 | Fees postponed indefinitely (secondary source only) | [novadata] |
| ~12 May 2026 | **Cancelled.** Amazon's page *"Cancellation of Third-Party Developer SP-API Fees"* says it "will not move forward with the SP-API usage and annual fees at this time", lets you delete a stored card, and removes the fee-preview dashboard. The page itself carries no date; the 12 May date comes from secondary reporting. | [fees-cancel] [novadata] |

**Bottom line:** today a private seller-only app costs nothing in API fees. "At this time" leaves room for a future proposal **[fees-cancel]**.

---

## 7. Amazon Ads API (separate from SP-API)

**Registration [ads-onb] [ads-apply]:**

- Separate from SP-API. Create an Amazon Developer account and a **Login with Amazon client**, then apply as a **Direct Advertiser** (the form is at advertising.amazon.com/partner-network/register-api), then assign API access to the LwA app.
- Review "may take up to 1 business day".
- The LwA-to-Ads-permissions association "cannot be changed once it is set" **[ads-apply]**.
- It's governed by the Ads API License Agreement and the Ads Data Protection Policy **[ads-onb]**.
- No fee for API access is mentioned in the onboarding docs (absence: **unverified**).

**Eligibility by ad type:**

| Ad type | Who can use it | Source |
|---|---|---|
| Sponsored Products | Professional sellers, vendors, KDP authors and agencies. The product must be in an eligible category and eligible for the Featured Offer. No Brand Registry mention. | [ads-sp-elig] |
| Sponsored Brands | "professional sellers enrolled in Amazon Brand Registry" (plus vendors etc.) | [ads-sb-elig] |
| Sponsored Display | Contextual/product targeting campaign types are for "seller brand owners and vendors" | [ads-sd] |

**Reporting surfaces [ads-rep]:**

| Surface | What | History | Limits | Status |
|---|---|---|---|---|
| **Reporting API v1** (new, "Amazon Ads v1") | Ad-hoc reports combining any dimensions, across ad products and accounts: campaign, ad group, ad, targeting, **search term**, advertised and purchased product, placement, plus metrics such as impressions, clicks, cost, purchases, sales, ROAS (7-day attribution for sellers) | **Daily data 24 months** (max 120-day range per report). Weekly 24 months. **Monthly and yearly 6 years.** Hourly 14 days. | **1 concurrent report per advertiser** (~100 reports/day); rate can be as low as 1 request/second | Current: generally available since 30 Sep 2026 **[ads-releases]** (Amazon's 8 Jun 2026 unified reporting announcement still calls it "Reporting API (beta)" **[ads-unified]**) | [ads-v1] [ads-v1-dates] [ads-v1-limits] [ads-unified] |
| **Reporting v3** | Report types: SP campaigns, targeting, search term, advertised product, purchased product; SB and SD equivalents; gross and invalid traffic | Retention: **SP 95 days** (SP search term 65); **SB 60 days** (SB purchased product 731); **SD 65 days**. Max 31 days per request. | 1–2 requests per advertiser per report type per day recommended. Up to 3 hours to generate. | **Shut off after 30 Jun 2027** | [ads-v3types] [ads-v3faq] [ads-v3-campaign] [ads-v3-st] [ads-v3-tgt] [ads-v3-ap] [ads-v3-pp] [ads-rep] |
| **Amazon Marketing Stream v2** | Push to your own S3 as Parquet, updated as often as every 15 minutes, matching the v1 Reporting API fields | Backfills up to 70 days when you subscribe | No polling or throttling | Recommended for recurring and large-scale reporting | [ads-rep] [ads-stream] |
| Exports | Campaign, ad group, ad and target **metadata** (budgets, status, bids) | Current state | — | — | [ads-rep] |
| **Amazon Ads MCP Server** | Official MCP server: Claude and other agents can query performance, list accounts and invoices, create reports and update campaigns | — | Expert tools (e.g. AMC SQL) in closed beta | — | [ads-mcp] |

**Data caveats:**

- Conversions restate for up to 60 days, so re-pull monthly **[ads-rep]**.
- The **API search-term report only includes terms with at least one click**, unlike the Ads console report **[ads-v3faq]**.

---

## 8. Rate limits and practical limits for "pull 12 months of everything"

Rates are Amazon's defaults per account–application pair. Some are dynamic and get raised for bigger sellers. The `x-amzn-RateLimit-Limit` response header shows your actual limit when it's present **[usage]**.

| Thing | Limit | Implication for a 12-month backfill | Source |
|---|---|---|---|
| `createReport` | 0.0167 req/s (≈1/min), burst 15 | Dozens of reports are fine; queue them | [reports-model] |
| `getReports` / `getReportSchedules` | 0.0222 req/s, burst 10 | Poll sparingly. Use the `REPORT_PROCESSING_FINISHED` notification instead. | [reports-model] [notif] |
| `getReportDocument` | 0.0167 req/s, burst 15 | — | [reports-model] |
| Report retention | 90 days max | **Settlement files older than ~90 days can't be fetched by API.** Rebuild that history from Finances ([`listTransactions`](transactions.md) / `listSummary` by settlement ID, or v0 event groups). | [reports-model] [rt-settle] |
| "Cancelled" reports | Some reports allow only 1 request per period; later requests are cancelled. Reports with no data are also cancelled. | Don't fire duplicates. Respect each report's cap. | [reports-faq] |
| [`GET_AMAZON_FULFILLED_SHIPMENTS_DATA_GENERAL`](fba_shipments.md) | **12 requests per 8 hours**, ≤1 month each | 12 months = 12 requests, i.e. one full 8-hour quota | [rt-fba] |
| Order tracking reports | Max **30-day** range each | 12–13 requests for 12 months | [rt-order] |
| FBA fee preview / referral fee preview / fee discounts | **1 per day** each | Daily snapshot only | [rt-fba] [rt-inv] [rt-ab] |
| [`GET_SALES_AND_TRAFFIC_REPORT`](sales_and_traffic.md) | **3 requests / 5 minutes**, ≤2 years back | One 12-month DAY report, or 12 monthly ones | [rt-analytics] |
| Inventory ledger detail | 18 months of history | Monthly slices | [rt-fba] |
| FBM returns reports | ≤60 days per report | 6–7 requests | [rt-ret] |
| **Orders v2026 `searchOrders`** | **0.0056 req/s (≈1 per 3 minutes), burst 20**. 100 orders per page. `paginationToken` expires after 24 hours. | For example, 20,000 orders = 200 pages ≈ 9 hours after the burst. For bulk history, prefer the order tracking reports. | [orders-rl] [orders-model] [orders-mig] |
| `getOrder` | 0.5 req/s, burst 30 | — | [orders-rl] |
| Orders history | **2 years max** | — | [orders] |
| Finances `listTransactions` | 0.5 req/s, burst 10. **A window over 180 days returns an empty response.** The last 48 hours may be missing. | 12 months = 3 windows, then paginate | [fin-model] |
| Finances v0 | 0.5 req/s, burst 30. 180-day windows. Group events only for the past 2 years. | — | [finv0-model] |
| Sales API `getOrderMetrics` | 0.5 req/s, burst 15. Hourly ≤30 days; other granularities ≤2 years. | One call per store | [sales-model] |
| FBA Inventory `getInventorySummaries` | 2 req/s, burst 2. Changed-since ≤18 months. | — | [fbainv-model] |
| Data Kiosk | Same rate limits as Reports. One running query per dataset per seller/app. Start ≤2 years back. | Run queries one after another | [dk] [dk-wf] [dk-sa] |
| Replenishment | Trailing 2 years | — | [replen-model] |
| Ads Reporting v1 | 1 concurrent report per advertiser | Run reports one at a time, or use Marketing Stream | [ads-v1-limits] |
| Ads v3 | SP 95 days, SB 60, SD 65 days retention; 31 days per request | Can't backfill 12 months on v3. Use v1 (24 months). | [ads-v3-campaign] |

---

## 9. Things the API explicitly CANNOT do (per Amazon's docs)

1. **Request or schedule settlement reports.** They're Amazon-generated only **[rt-settle]**. The API also can't list them beyond the 90-day report retention **[reports-model]**.
2. **Return orders older than 2 years** (except JP pre-orders) **[orders]**.
3. **Give you every Seller Central report.** "Seller Central provides additional reports that you can't access using the Reports API" **[reports-faq]**.
4. **Read buyer–seller messages.** The Messaging API only sends messages; its only read operations list available actions and attributes **[messaging] [rmap]**.
5. **Generate some reports by API:**
   - The US Sales Tax report must be generated in Seller Central first **[rt-tax]**.
   - The Deferred Transaction report must be requested in the Seller Central Payments Reports Repository first **[rt-pay]**.
6. **Schedule or backlog Data Kiosk queries server-side** **[dk]**.
7. **Give buyer email or phone for FBA orders through the Orders API** in UK/EU, even with restricted roles **[orders-pii]**. The Amazon Fulfilled Shipments report ([fba_shipments](fba_shipments.md)) is the exception: its `buyer-email` column can hold the buyer's email on UK orders.
8. **Give Brand Analytics search/basket/repeat reports or Sponsored Brands without Brand Registry** **[rt-analytics] [ads-sb-elig]**.
9. **Use these APIs in the UK:** Easy Ship **[easyship]**, AWD (US only) **[awd]**, the Tracking API (US only) **[tracking]**. The FBA Promotions report is NA only and the Replacements report NA/IN only **[rt-fba]**.
10. **Give FBA returns in the "Returns" reports group.** Those reports cover seller-fulfilled returns only **[rt-ret]**.
11. **Serve the Data Kiosk Seller Analytics B2B metrics.** They're declared but "not yet supported" **[dk-sa]**.
12. **Include zero-click search terms in the Ads API search-term report** **[ads-v3faq]**.
13. **Give Seller Wallet data without a licensed-TPP role** (AISP/PISP) **[tpp] [wallet]**. Payment methods are the exception: Transfers `getPaymentMethods` works with Finance and Accounting in the EU region (see Transfers above).
14. **Return anything in the V2 performance report's "warning states".** The field is deprecated and always empty **[rt-perf]**.
15. **Customer Feedback API limits:** refreshed weekly, English only, and return insights only at browse-node level, not per ASIN **[cf]**.

---

## 10. Unverified / not found

- **Brand Analytics role for a reseller who isn't the brand owner:** whether Amazon will grant it to a private app (needed for Sales & Traffic and Data Kiosk). The docs don't state a Brand Registry requirement for the role itself.
- **Restricted-role approval odds and timelines** for private apps aren't published.
- **History depth for some reports isn't documented:** reimbursements, FBA returns, removals, storage fees, feedback, V1/V2 performance, ledger summary. Order tracking reports say "start date can be any date", but depth beyond 2 years is untested.
- **The private-seller fee exemption** is confirmed only by secondary sources; Amazon's original fee page is gone (404). The cancellation itself is on Amazon's page **[fees-cancel]**.
- **No Vine report or API**, and no appeals / Plan-of-Action API, was found (absence only).
- **Ads API access fee:** none mentioned (absence only).
- **A+ Content Brand Registry requirement:** not confirmed from the API docs.

---

## References

SP-API docs base: `https://developer-docs.amazon/sp-api/docs/` (the old `developer-docs.amazon.com/sp-api/docs/` redirects there)

- [models] https://developer-docs.amazon/sp-api/docs/sp-api-models
- [llms] https://developer-docs.amazon/sp-api/llms.txt
- [roles] https://developer-docs.amazon/sp-api/docs/roles-in-the-selling-partner-api
- [rmap] https://developer-docs.amazon/sp-api/docs/role-mappings
- [rtypes] https://developer-docs.amazon/sp-api/docs/role-mappings-for-types
- [deprec] https://developer-docs.amazon/sp-api/docs/sp-api-deprecations
- [relnotes] https://developer-docs.amazon/sp-api/docs/sp-api-release-notes
- [rt] https://developer-docs.amazon/sp-api/docs/report-type-values
- [rt-fba] https://developer-docs.amazon/sp-api/docs/report-type-values-fba
- [rt-order] https://developer-docs.amazon/sp-api/docs/report-type-values-order
- [rt-analytics] https://developer-docs.amazon/sp-api/docs/report-type-values-analytics
- [rt-perf] https://developer-docs.amazon/sp-api/docs/report-type-values-performance
- [rt-settle] https://developer-docs.amazon/sp-api/docs/report-type-values-settlement
- [rt-tax] https://developer-docs.amazon/sp-api/docs/report-type-values-tax
- [rt-inv] https://developer-docs.amazon/sp-api/docs/report-type-values-inventory
- [rt-ret] https://developer-docs.amazon/sp-api/docs/report-type-values-returns
- [rt-reg] https://developer-docs.amazon/sp-api/docs/report-type-values-regulatory-compliance
- [rt-invoice] https://developer-docs.amazon/sp-api/docs/report-type-values-invoice-data
- [rt-pay] https://developer-docs.amazon/sp-api/docs/report-type-values-payment
- [rt-ab] https://developer-docs.amazon/sp-api/docs/report-type-values-amazon-business
- [rt-es] https://developer-docs.amazon/sp-api/docs/report-type-values-easy-ship
- [rt-b2b] https://developer-docs.amazon/sp-api/docs/report-type-values-b2b-product-opportunities
- [rt-bt] https://developer-docs.amazon/sp-api/docs/report-type-values-browse-tree
- [reports-model] https://github.com/amzn/selling-partner-api-models/blob/main/models/reports-api-model/reports_2021-06-30.json
- [reports-faq] https://developer-docs.amazon/sp-api/docs/reports-faq
- [auto-reports] https://developer-docs.amazon/sp-api/docs/retrieve-automatically-generated-reports
- [orders] https://developer-docs.amazon/sp-api/docs/orders-api
- [orders-mig] https://developer-docs.amazon/sp-api/docs/orders-api-migration-guide
- [orders-pii] https://developer-docs.amazon/sp-api/docs/access-orders-pii
- [orders-rl] https://developer-docs.amazon/sp-api/docs/orders-api-rate-limits
- [orders-model] https://github.com/amzn/selling-partner-api-models/blob/main/models/orders-api-model/orders_2026-01-01.json
- [fin] https://developer-docs.amazon/sp-api/docs/finances-api
- [fin-model] https://github.com/amzn/selling-partner-api-models/blob/main/models/finances-api-model/finances_2024-06-19.json
- [finv0-model] https://github.com/amzn/selling-partner-api-models/blob/main/models/finances-api-model/financesV0.json
- [transfers] https://developer-docs.amazon/sp-api/docs/transfers-api
- [wallet] https://developer-docs.amazon/sp-api/docs/seller-wallet-api
- [tpp] https://developer-docs.amazon/sp-api/docs/tpp-registration
- [sellers] https://developer-docs.amazon/sp-api/docs/sellers-api
- [sellers-model] https://github.com/amzn/selling-partner-api-models/blob/main/models/sellers-api-model/sellers.json
- [sales] https://developer-docs.amazon/sp-api/docs/sales-api
- [sales-model] https://github.com/amzn/selling-partner-api-models/blob/main/models/sales-api-model/sales.json
- [support] https://developer-docs.amazon/sp-api/docs/support-api
- [promos] https://developer-docs.amazon/sp-api/docs/promotions-api
- [cf] https://developer-docs.amazon/sp-api/docs/customer-feedback-api
- [replen] https://developer-docs.amazon/sp-api/docs/replenishment-api
- [replen-model] https://github.com/amzn/selling-partner-api-models/blob/main/models/replenishment-api-model/replenishment-2022-11-07.json
- [dk] https://developer-docs.amazon/sp-api/docs/data-kiosk-api
- [dk-sa] https://developer-docs.amazon/sp-api/docs/seller-analytics-dataset-guide
- [dk-sched] https://developer-docs.amazon/sp-api/docs/schedule-data-kiosk-queries
- [dk-wf] https://developer-docs.amazon/sp-api/docs/data-kiosk-workflow-guide
- [dk-schemas] https://github.com/amzn/selling-partner-api-models/tree/main/schemas/data-kiosk
- [dk-econ] https://github.com/amzn/selling-partner-api-models/blob/main/schemas/data-kiosk/analytics_economics_2024_03_15.graphql
- [dk-roles] https://developer-docs.amazon/sp-api/changelog/correction-data-kiosk-role-requirements
- [dk-st] https://github.com/amzn/selling-partner-api-models/blob/main/schemas/data-kiosk/analytics_salesAndTraffic_2024_04_24.graphql
- [notif] https://developer-docs.amazon/sp-api/docs/notification-type-values
- [rdt] https://developer-docs.amazon/sp-api/docs/authorization-with-the-restricted-data-token
- [tokens] https://developer-docs.amazon/sp-api/docs/tokens-api
- [data-access] https://developer-docs.amazon/sp-api/docs/amazon-seller-data-access
- [reg] https://developer-docs.amazon/sp-api/docs/sp-api-registration-overview
- [reg-priv] https://developer-docs.amazon/sp-api/docs/register-as-a-private-developer
- [selfauth] https://developer-docs.amazon/sp-api/docs/self-authorization
- [authlimits] https://developer-docs.amazon/sp-api/docs/application-authorization-limits
- [credrot] https://developer-docs.amazon/sp-api/docs/set-up-credential-rotation-notifications
- [lwarot] https://developer-docs.amazon/sp-api/docs/rotating-your-apps-lwa-credentials
- [appmgmt] https://developer-docs.amazon/sp-api/docs/application-management-api
- [appint] https://developer-docs.amazon/sp-api/docs/app-integrations
- [usage] https://developer-docs.amazon/sp-api/docs/usage-plans-and-rate-limits
- [fbainv] https://developer-docs.amazon/sp-api/docs/fba-inventory-api
- [fbainv-model] https://github.com/amzn/selling-partner-api-models/blob/main/models/fba-inventory-api-model/fbaInventory.json
- [inbound] https://developer-docs.amazon/sp-api/docs/fulfillment-inbound-api
- [outbound] https://developer-docs.amazon/sp-api/docs/fulfillment-outbound-api
- [awd] https://developer-docs.amazon/sp-api/docs/amazon-warehousing-and-distribution-api
- [easyship] https://developer-docs.amazon/sp-api/docs/easy-ship-api
- [pricing] https://developer-docs.amazon/sp-api/docs/product-pricing-api
- [fees] https://developer-docs.amazon/sp-api/docs/product-fees-api
- [listings] https://developer-docs.amazon/sp-api/docs/listings-items-api
- [restrictions] https://developer-docs.amazon/sp-api/docs/listings-restrictions-api
- [catalog] https://developer-docs.amazon/sp-api/docs/catalog-items-api
- [ptd] https://developer-docs.amazon/sp-api/docs/product-type-definitions-api
- [autoprice] https://developer-docs.amazon/sp-api/docs/manage-automated-pricing-rules
- [messaging] https://developer-docs.amazon/sp-api/docs/messaging-api
- [solicit] https://developer-docs.amazon/sp-api/docs/solicitations-api
- [mfn] https://developer-docs.amazon/sp-api/docs/merchant-fulfillment-api
- [services] https://developer-docs.amazon/sp-api/docs/services-api
- [supply] https://developer-docs.amazon/sp-api/docs/supply-sources-api
- [vehicles] https://developer-docs.amazon/sp-api/docs/vehicles-api
- [tracking] https://developer-docs.amazon/sp-api/docs/tracking-api
- [invoices] https://developer-docs.amazon/sp-api/docs/invoices-api
- [feeds] https://developer-docs.amazon/sp-api/docs/feeds-api
- [policies] https://developer-docs.amazon/sp-api/docs/policies-and-agreements
- [dpp] https://sellercentral.amazon.com/mws/static/policy?documentType=DPP&locale=en_GB
- [aup] https://sellercentral.amazon.com/mws/static/policy?documentType=AUP&locale=en_GB
- [fees-cancel] https://developer.amazonservices.com/cancellation-of-sp-api-fees
- [gh5025] https://github.com/amzn/selling-partner-api-models/discussions/5025 (developer discussion quoting the Nov 2025 announcement; secondary)
- [ppcland] https://ppc.land/amazon-introduces-fees-for-third-party-developer-api-access-in-2026/ (secondary)
- [novadata] https://novadata.io/resources/news/amazon-sp-api-subscription-fees-2026 (secondary; vendor with commercial interest)

Amazon Ads docs. Markdown mirror base: `https://d3a0d0y2hgofx6.cloudfront.net/en-us/`, named as the official raw source in Amazon's llms.txt [ads-llms].

- [ads-llms] https://advertising.amazon.com/API/docs/en-us/llms.txt
- [ads-rep] https://d3a0d0y2hgofx6.cloudfront.net/en-us/guides/reporting/overview.md
- [ads-v1] https://d3a0d0y2hgofx6.cloudfront.net/en-us/guides/reporting/ads-v1/overview.md
- [ads-v1-dates] https://d3a0d0y2hgofx6.cloudfront.net/en-us/guides/reporting/ads-v1/dates.md
- [ads-v1-limits] https://d3a0d0y2hgofx6.cloudfront.net/en-us/guides/reporting/ads-v1/limits.md
- [ads-unified] https://advertising.amazon.com/resources/whats-new/streamline-campaign-analysis-with-unified-reporting
- [ads-releases] https://advertising.amazon.com/API/docs/en-us/release-notes/index
- [ads-v3types] https://d3a0d0y2hgofx6.cloudfront.net/en-us/guides/reporting/v3/report-types/overview.md
- [ads-v3faq] https://d3a0d0y2hgofx6.cloudfront.net/en-us/guides/reporting/v3/faq.md
- [ads-v3-campaign] https://d3a0d0y2hgofx6.cloudfront.net/en-us/guides/reporting/v3/report-types/campaign.md
- [ads-v3-st] https://d3a0d0y2hgofx6.cloudfront.net/en-us/guides/reporting/v3/report-types/search-term.md
- [ads-v3-tgt] https://d3a0d0y2hgofx6.cloudfront.net/en-us/guides/reporting/v3/report-types/targeting.md
- [ads-v3-ap] https://d3a0d0y2hgofx6.cloudfront.net/en-us/guides/reporting/v3/report-types/advertised-product.md
- [ads-v3-pp] https://d3a0d0y2hgofx6.cloudfront.net/en-us/guides/reporting/v3/report-types/purchased-product.md
- [ads-stream] https://d3a0d0y2hgofx6.cloudfront.net/en-us/guides/amazon-marketing-stream/v2/overview.md
- [ads-mcp] https://d3a0d0y2hgofx6.cloudfront.net/en-us/mcp/mcp-overview.md
- [ads-onb] https://d3a0d0y2hgofx6.cloudfront.net/en-us/guides/onboarding/overview.md
- [ads-apply] https://d3a0d0y2hgofx6.cloudfront.net/en-us/guides/onboarding/apply-for-access.md
- [ads-sd] https://d3a0d0y2hgofx6.cloudfront.net/en-us/guides/sponsored-display/overview.md
- [ads-sb-elig] https://advertising.amazon.com/solutions/products/sponsored-brands
- [ads-sp-elig] https://advertising.amazon.com/solutions/products/sponsored-products
