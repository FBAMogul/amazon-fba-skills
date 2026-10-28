# Seller Central UK: what you can see and download vs what the API gives you

Researched 18 Oct 2026 from Amazon's UK help pages and API docs. The menus were then checked live in Seller Central UK on 28 Oct 2026, with some of the screens and their downloads (see "The live menus" below). The API column for each screen still comes from the 18 Oct research. Screens the live check didn't open are described as the help pages describe them.

This is written for a UK FBA seller's account under the Amazon Services Europe agreement (amazon.co.uk, and possibly EU stores). It starts from the Seller Central screens and asks: which of them have no API route? The SP-API itself is catalogued in [report-catalogue](report-catalogue.md), so it is covered here only where it answers that question. Report types that have their own page in this pack link to it.

**How it was checked.** The research used Amazon's current SP-API docs (developer-docs.amazon.com now redirects to developer-docs.amazon), the SP-API model files on GitHub, and the UK Seller Central help pages. The help pages render through JavaScript, so they were read through the same public JSON endpoint the help page itself calls; every UK help citation below is the normal public URL for that page. Third-party sources are named as such, and anything that couldn't be confirmed is marked **[unverified]**. "No API route found" means the full SP-API models list [MODELS], every report type [RTV], the notification types [NOTIF] and the 2025–2026 release notes [RN] were checked and nothing was found. It does not prove that no route exists.

---

## 0. Key findings (read this first)

1. **Several "UI-only" areas now have API routes, all new in 2025–2026:**
   - **Case log:** the Support API v2025-02-01 (launched 30 Sep 2026) lists your support cases with their details, status and full email, phone and chat history. It is read-only: you can't open or reply to a case through it. It needs the restricted "Selling Partner Support" role. [SUPPORT] [RN]
   - **Payments reserve and balances:** Finances API `listBalances` (29 Jul 2026) returns AVAILABLE, RESERVED, DEFERRED, ACCOUNT_LEVEL_RESERVE and TOTAL balances. [BAL] [RN]
   - **Payouts:** Transfers API `listPayouts` and `listExpectedPayouts` (29 Jul 2026). [TRANSFERS] [RN]
   - **Account health metrics:** [`GET_V2_SELLER_PERFORMANCE_REPORT`](account_health.md) gives the AHR score and status plus counts per policy category [RTV-PERF]. The Data Kiosk Seller Analytics dataset (30 Sep 2026; pack page [seller_analytics](seller_analytics.md)) adds daily, weekly or monthly account-health metrics such as ODR, A-to-z, chargeback and negative-feedback counts and rates, the AHR score and 120-day product-defect rates. [SA-DK] [SA-SCHEMA]
   - **Deals, coupons and promotions:** the Promotions API 2025-12-01 (26 Aug 2026) can read promotions; the coupon and promotion performance reports were already available. [PROMO] [RTV-PERF]
   - **Review and return insights:** the Customer Feedback API (Jun 2025, UK included). Amazon says it gives "the same information as the Product Opportunity Explorer" for review and return topics. [CFA]
   - **Inventory planning and inventory age:** [`GET_FBA_INVENTORY_PLANNING_DATA`](inventory_planning.md), the restock report, the stranded report and the storage-fee report (which includes storage-utilisation fields). [RTV-FBA]
2. **No API was found for capacity limits or the IPI score.** FBA capacity limits and the IPI score still look UI-only. Amazon's UK help says limits are shown "in the Capacity Monitor at the bottom of the Fulfilment by Amazon dashboard" and announced by email [H-CAP]. Neither appears in the SP-API models, report types, release notes or the Data Kiosk schema, and a GitHub code search of the SP-API models found no capacity or IPI fields.
3. **The real gaps still worth reading through the browser:**
   - the detail behind account health: the list of individual policy violations by ASIN, the notice text and appeal status
   - the Performance Notifications inbox
   - the Buyer-Seller Messages inbox (the Messaging API can only send)
   - the status and history of ungating applications
   - capacity limits and Capacity Manager
   - the IPI score
   - Voice of the Customer (NCX rate, CX Health and the comments)
   - claim-level detail for A-to-z claims, chargebacks and SAFE-T claims (beyond a SAFE-T claim's id and state)
   - Amazon Lending offers and loan terms
   - Seller fee VAT invoices in the Tax Document Library
   - User permissions and notification settings
   - Automate Pricing rules
   - Brand-only tools (experiments, Vine, the brand dashboard), which mostly don't apply to a reseller who isn't the brand owner
4. **History limits matter for a 12-month pull.**
   - The Reports API keeps a generated report for at most 90 days [GETREPORTS]. Settlement reports are created by Amazon and can't be requested [RTV-SET], so in practice settlement reports older than about 90 days won't come through the API **[inference]**. Rebuild that history from the Finances API (180-day windows [TXN]) or from Date Range reports in the UI, which go back to 2012 [H-DRR].
   - Ads history depends on the report type. The Ads console's older Sponsored Ads reports each have their own look-back: purchased product 60 days, search term 65, advertised product, targeting, placement and Performance Over Time 90, gross and invalid traffic 365 days, budget 2 years and the Sponsored Products campaign report 5 years (30 days by hour) [ADS-SP-PP] [ADS-SP-CAMPAIGN] ([ads_reports](ads_reports.md) links Amazon's page for each). From 17 Dec 2026 no new reports can be created or edited on those pages, and on 31 Dec 2026 Amazon shuts them and everything still on them is "permanently deleted" [ADS-MIGRATE]. Ads API v3 keeps 60–95 days for most reports, and 731 days for the Sponsored Brands purchased product report ([report-catalogue](report-catalogue.md) §7). Unified reporting in the Ads console (Measurement and Reporting > Reporting), generally available since 8 Jun 2026 with the UK included [ADS-UNIFIED], keeps 24 months of daily or weekly data and 6 years (72 months) of monthly, yearly or summary data; one report can cover at most 120 days by day, 15 months by week or 25 months by month [ADS-DATA] [ADS-UR] [ADS-UR-CREATE]. Amazon raised daily and weekly history from 15 to 24 months on 26 Aug 2026; the 8 Jun 2026 announcement still says 15. The longest range one report can cover didn't change [ADS-API-RELEASES]. Through the Ads API, the v1 Reporting API, generally available since 30 Sep 2026 [ADS-API-RELEASES], keeps 24 months of daily data and 6 years of monthly ([report-catalogue](report-catalogue.md) §7).
   - The detailed inventory ledger covers 18 months [H-LEDGER].
   - Business Reports cover about 2 years [H-BR].
5. **Browser automation rules differ by marketplace.**
   - **US/CA (North America):** since 4 Mar 2026 the BSA has a "Use of Agents" section 19, and a separate Agent Policy requires any automated agent to put "Agent/[agent name]" in its user-agent string, not mimic human keystrokes, and not solve CAPTCHAs [BSA-US] [AGENT-US].
   - **UK/EU:** the Amazon Services Europe BSA your account runs under has no Agent clause and no robots/data-mining clause as of today [BSA-UK] [BSA-UK-CHANGES]. The Agent Policy page doesn't exist on the UK help site.
   - **However:** the amazon.co.uk Conditions of Use (28 Nov 2025) carry the same Agent Terms for "Amazon Services" [COU-UK]. Amazon's UK help also says account credentials "should not be shared with anyone" [H-PERMS], and the AUP that binds API developers bans asking for seller passwords and says Seller Central access should come through user permissions [AUP].
   - **Bottom line:** an agent reading your own Seller Central in your own logged-in Chrome isn't clearly banned under the UK BSA text, but it is a grey area. It would plainly fail the US Agent Policy's technical rules for an account that covers US/CA. Detail in section 4.

---

## The live menus (checked 28 Oct 2026)

Checked live in Seller Central UK on 28 Oct 2026.

- **Two layouts.** Seller Central UK now has the "New Amazon selling experience", with the home page at `/amazonsell/business`. Its ☰ menu is listed below. At the foot of the menu are a **Site map** link and a **Classic Seller Central** link back to the old layout.
- **The classic layout is going.** Some accounts see a banner in it saying that soon, switching back to it as your default "will no longer be available".
- **Business Reports is going too.** Some accounts see a banner on its page telling sellers to switch to Custom Analytics "by year-end". It isn't in the new ☰ menu: open it from **Site map > Other resources > Business reports**.
- **Develop Apps is only in the classic menu.** The new ☰ menu and the Site map have no entry for it. See the end of this section.
- **Your menus may differ.** Items tied to a programme, store, category or user permission (for example brand tools, B2B, AWD, Pan-EU, automotive fitment, Lending and VAT services) may be missing from some accounts' menus. Read these lists as where pages sit, not as what an account has.
- The tables in section 1 are grouped roughly by the classic menu's headings. Use the lists here to find a page in the new layout.

### New ☰ menu

- **My business**: the home page. Its top right has an "All reports" drop-down.
- **Products**: Workspace; Manage products (`/amazonsell/manage-products`); Manage pricing (`/pricing/health`); Add products (`/product-search`); Analyse reviews (`/feedback-manager/index.html`, with Feedback Manager and Voice of Customer tabs); Research products (`/new-product-guidance`); Sell Globally; Product reports (`/listing/reports`).
- **Supply chain**: Workspace; Manage Amazon inventory (`/amazonsell/fba-inventory`, the FBA Inventory page); Manage global store; Manage Seller Fulfilled inventory; Manage shipments (`/manage/shipments`); Send to Amazon (`/fba/sendtoamazon`); Manage channel destinations; Supply chain console; Supply chain reports (`/reportcentral/WelcomePage`, the Amazon Fulfilment reports).
- **Orders**: Workspace; Manage orders (`/amazonsell/orders/mfn`); Manage quotes; Manage returns (`/manage/returns/mfn`); Manage claims (`/gp/chargebacks/home.html`); Order reports (`/order-reports-and-feeds/reports`).
- **Marketing**: Workspace; Manage promotions and deals (`/promotion-central/dashboard`); Ads console (`/cm`); Vine (`/vine`).
- **Finance**: Workspace; Manage payments (`/payments/dashboard/index.html`); Manage taxes; Profit analytics (`/economics`); Finance reports (`/payments/reports-repository`, with Payment reports and Tax document library tabs).
- **Account health**: Manage account health (`/performance/dashboard`).
- **Programs and apps**: Explore programmes; Selling partner appstore; Explore services.
- At the foot: Site map (`/sitemap`); Classic Seller Central.

### Site map (`/sitemap`)

Amazon describes the Site map as all the tools and pages in the new Seller Central. It uses the ☰ menu's headings and adds Settings, Support resources and Other resources. These are the pages it has that the ☰ menu doesn't:

- **Products**: Manage A+ content; Manage images; Manage videos; View selling applications (`/hz/myqdashboard`); Automate pricing (`/automatepricingv2/pricingrules`); Flexible financing; Voice of Customer (`/voice-of-the-customer`); Marketplace product guidance; Feedback report (`/feedback-manager/index.html#/report`).
- **Supply chain**: Manage AWD inventory; Manage stranded inventory (`/inventoryplanning/stranded-inventory`); Manage unfulfillable inventory (`/myinventory/unfulfillable/index.html`); Send to AWD; Move from AWD; under Supply chain reports, Seller fulfilled reports and FBA inventory reports.
- **Orders**: Create MCF order; Upload order files; Returns and recovery insights; Manage FBA returns (`/fba-return`); Manage A-to-z Claims (`/gp/guarantee-claims/home.html`); Manage Safe-T claims (`/safet-claims`); under Order reports, Seller fulfilled return reports (`/returns/report`), FBA customer return reports (`/reportcentral/CUSTOMER_RETURNS/0`) and Replacement reports.
- **Marketing**: Manage stores.
- **Finance**: Lending (`/lending/ph/offers`); Seller Wallet (`/sellerwallet`); Currency converter; Manage VAT (`/service-hub`); File VAT (`/vat-services/filing-dashboard`); Revenue calculator (`/revcal`); Selling economics (`/sereport`); under Finance reports, Payment reports (`/payments/reports-repository`) and Tax document library (`/gp/tax/tax-library.html`).
- **Account health**: Performance notifications (`/performance/notifications`); Manage compliance (`/performance/account/health/product-policies`).
- **Settings**: User permissions (`/account/permissions`); Account information (`/hz/sc/account-information`); Account settings; Notification preferences (`/notifications/preferences`); Login settings; Return settings; Gift options; Delivery settings; Tax settings; FBA settings (`/fba/settings/index.html`); Supply chain settings; Premium services settings; Manage your apps (`/apps/manage`, the apps you've authorised); Manage your certifications; Manage your brands; Multichannel settings.
- **Support resources**: Get help and resources; Manage support cases (`/cu/case-lobby`); Strategic account services; Seller University.
- **Other resources**: Business reports (`/business-reports#/dashboard`); B2B Central; Custom analytics (`/custom-analytics/…`); Virtual bundles; Brand transparency; Buyer seller messaging (`/messaging/inbox-v3`).

### Classic menu

Open it from **Classic Seller Central** at the foot of the new ☰ menu. Its top bar has a "New Seller Central" switch to go back. Hover over a heading to see its items.

- **Catalogue**: Add Products; Add Products via Upload; Complete Your Drafts; View Selling Applications; Upload Images; Upload and Manage Videos.
- **Inventory**: Manage All Inventory (`/myinventory/inventory`); Manage seller-fulfilled products; Sell Globally; Manage Automotive Fitment Data; Automotive Fitment; FBA: Dashboard (`/fba/dashboard`), FBA Inventory (`/inventoryplanning/manageinventoryhealth`), Shipments, Opportunities, Analytics; Remote Fulfillment with FBA; Manage Pan-EU FBA Inventory; Warehousing and Distribution (AWD).
- **Pricing**: Pricing Health; Automate Pricing; Fee Discounts; Enable no fees installments.
- **Orders**: Manage Orders; Create MCF Order; Order Reports; Manage Invoicing; Upload Order Related Files; Insights and Opportunities; Manage Seller Fulfilled Returns; Manage SAFE-T Claims.
- **Advertising**: Campaign Manager; A+ Content Manager; Vine; Deals; Vouchers; Price Discounts; Manage Promotions.
- **Stores**: Manage Stores.
- **Growth**: Manage Buy with Prime; Growth Opportunities; Explore programmes; Custom Programme; Marketplace Product Guidance; Lending; Product Opportunity Explorer; Multi-Channel Fulfilment (MCF); Sale Event Planner.
- **Reports**: Business Reports; Custom analytics; Fulfilment by Amazon (`/reportcentral`); Advertising Reports (opens the Ads console); Return Reports (`/returns/report`); Custom Reports (`/customizedReports/lp`); Inventory Reports (`/listing/reports`); Tax Document Library; Manage Taxes; Manage your VAT; Manage your EPR Compliance; Selling economics and fees; SKU Economics report (`/sereport`).
- **Payments**: Payments; Reports Repository; Seller Wallet; Currency Converter.
- **Performance**: Account Health; Feedback Manager; A-to-z Guarantee Claims; Chargeback Claims; Performance Notifications; Voice of the Customer.
- **Apps and Services**: Selling Partner Appstore; Manage Your Apps; Explore Services; Manage Service requests; Develop Apps (`/sellingpartner/developerconsole`).
- **B2B**: B2B Central; Product Opportunities; Manage Quotes; Business Profile; Certifications; Business discount insights; Case pack recommendations.
- **Brands**: Manage Your Brands; Brand Protection.
- **Learn**: Seller University; Forums; News.

**Develop Apps** shows a message that you're being redirected to the Solution Provider Portal, Amazon's portal for developers and service providers. The portal (solutionproviderportal.amazon.com) then asks you to sign in again with your Seller Central login. The nearest page in the new layout is **Settings > Manage your apps**, but that only lists apps you've already authorised. [getting-access](getting-access.md) covers registering as a developer.

### Downloading by hand

[seller-central-downloads](seller-central-downloads.md) covers the downloads themselves: file names, formats, date styles on the forms and in the files, and the traps they share.

---

## 1. The Seller Central UK menu, mapped

Legend:
- **API = yes**: a documented SP-API or Ads API route covers the data.
- **Partial**: some of it is covered.
- **No API route found**: as defined in the method note above.

The tables are grouped roughly by the classic menu's headings. A path in brackets in the Screen column is where the page sits in the new layout, as seen in the live check on 28 Oct 2026; one starting "Site map" is only in the Site map. Downloads seen in the live check are added to the UI download column.

### 1.1 Catalogue / Inventory

| Screen | What it shows | UI download | API route | Sources |
|---|---|---|---|---|
| Manage All Inventory / listings (Products > Manage products; Products > Product reports) | Your SKUs, price, quantity, status | Product reports (`/listing/reports`, titled "Inventory reports"): choose a report under "Request an Inventory report", such as All Listings or Active Listings. All Listings comes as txt only and "can take up to 45 minutes" | **Yes**: Listings Items API; [`GET_MERCHANT_LISTINGS_ALL_DATA`](listings_all.md) and related reports | [MODELS] [RTV-INV] [H-MAI] |
| Fix suppressed listings | Suppressed listings with the reason | Report | **Yes**: [`GET_MERCHANTS_LISTINGS_FYP_REPORT`](suppressed_listings.md) ("suppressed listings, the reason… and instructions"); Listings Items API issues; `LISTINGS_ITEM_ISSUES_CHANGE` notification | [RTV-INV] [NOTIF] |
| Manage FBA Inventory / FBA Inventory page (Supply chain > Manage Amazon inventory) | Available, inbound, reserved and researching stock, sell-through, estimated storage cost | Reports drop-down: Inventory report and Restock report (csv/txt). Also a Download Demand Forecast button | **Yes**: FBA Inventory API; `GET_FBA_MYI_UNSUPPRESSED_INVENTORY_DATA`, [`GET_FBA_MYI_ALL_INVENTORY_DATA`](inventory_all.md), [`GET_FBA_INVENTORY_PLANNING_DATA`](inventory_planning.md), [`GET_RESERVED_INVENTORY_DATA`](reserved_inventory.md) | [RTV-FBA] [H-FBAINV] [H-MFI] |
| Inventory Planning / Manage inventory health / Inventory Age | Aged stock, aged inventory surcharge estimates, excess stock, historical days of supply | Yes (FBA Inventory report includes `estimated-ais-…` columns). Amazon's page says this report replaces the Inventory Health, Inventory Age and Excess Inventory reports | **Yes**: `GET_FBA_INVENTORY_PLANNING_DATA` ("FBA Manage Inventory Health"). A March 2026 GitHub thread says the Jan 2026 aged-inventory bucket changes weren't yet in the API **[third-party]** | [RTV-FBA] [H-FBAINVRPT] [GH-5179] |
| Restock Inventory | Recommended reorder quantity and dates | Restock report; also a Restock tab on the FBA Inventory page | **Yes**: [`GET_RESTOCK_INVENTORY_RECOMMENDATIONS_REPORT`](restock.md) | [RTV-FBA] [H-RESTOCK] |
| Capacity limits (Capacity Monitor; in the live check, a drawer at the bottom of the FBA Inventory page) | Monthly limit in cubic feet, usage, estimates for the next 2 months | Not described as a download | **No API route found.** UK help: limits are shown in the Capacity Monitor and announced by email in the fourth week of the month. The storage-fee report has `storage_utilization_ratio` fields but no limit | [H-CAP] [RTV-FBA] [FORUM-CAP] |
| Capacity Manager | Requests for extra capacity, reservation fees, performance credits | "Download calculator tool" and "Download reservation fees history" | **No API route found** | [H-CAPMGR] |
| Inventory Performance / IPI | IPI score and its parts (excess, sell-through, stranded, in-stock) | Not described | **No API route found** for the score. Some of its parts appear in the inventory planning and stranded reports **[inference]** | [H-IPI] [H-INVPERF] [RTV-FBA] |
| Stranded inventory (Site map > Supply chain > Manage stranded inventory) | Stranded units and the reasons | Yes: Stranded Inventory and Bulk Fix Stranded Inventory in Supply chain reports | **Yes**: [`GET_STRANDED_INVENTORY_UI_DATA`](stranded.md), `GET_STRANDED_INVENTORY_LOADER_DATA` | [RTV-FBA] [H-STRANDED] |
| Removals | Recommended removals, removal orders and shipments | Removal Order Detail and Removal Shipment Detail in Supply chain reports. That list may not include a Recommended removals report | **Yes** (reports): [`GET_FBA_RECOMMENDED_REMOVAL_DATA`](recommended_removals.md), [`…REMOVAL_ORDER_DETAIL_DATA`](removal_orders.md), [`…REMOVAL_SHIPMENT_DETAIL_DATA`](removal_shipments.md). Removal orders can be created through the feed `POST_FLAT_FILE_FBA_CREATE_REMOVAL` **[from a search snippet of Amazon's feed-type page]** | [RTV-FBA] [H-REMOVAL] [FEEDS-FBA] |
| Shipments (Supply chain > Manage shipments; Supply chain > Send to Amazon) | Inbound plans and shipments | Inbound Performance report in Supply chain reports | **Yes**: Fulfillment Inbound API v2024-03-20; inbound non-compliance report | [MODELS] [RTV-FBA] |
| FBA cost report (EU customs) | Customs value for non-EU to EU shipments | PDF download in Send to Amazon | **No API route found** (only matters for imports into the EU) | [H-FBACOST] |

### 1.2 Pricing

| Screen | What it shows | UI download | API route | Sources |
|---|---|---|---|---|
| Manage Pricing / Pricing Health (Products > Manage pricing) | Prices, Featured Offer eligibility warnings | — | **Yes**: Product Pricing API; `PRICING_HEALTH` and `ANY_OFFER_CHANGED` notifications | [MODELS] [NOTIF] |
| Fee Preview | Estimated selling and FBA fees per SKU ("may be up to 72 hours old") | csv/txt, in Supply chain reports | **Yes**: [`GET_FBA_ESTIMATED_FBA_FEES_TXT_DATA`](fba_fee_preview.md); Product Fees API; `GET_REFERRAL_FEE_PREVIEW_REPORT` | [H-FEEPREV] [RTV-FBA] [RTV-INV] |
| Automate Pricing (Site map > Products > Automate pricing) | Your repricing rules and their price history | — | **Partial**: the rule names and ids (Product Type Definitions `getDefinitionsProductType`) and each SKU's rule and price limits (Listings Items `getListingsItem`), as [not-in-the-api](not-in-the-api.md) explains. **No API route found** for the rules' settings or what they did. You can still change prices through the Listings or Feeds APIs | [H-AUTOPRICE] [AUTOPRICE-API] [MODELS] |

### 1.3 Orders

| Screen | What it shows | UI download | API route | Sources |
|---|---|---|---|---|
| Manage Orders / order reports (Orders > Manage orders; Orders > Order reports) | Orders; FBM order reports; All Orders; Archived Orders (over 2 years old, no buyer PII) | txt. Order reports tabs: FBM orders, FBM unshipped orders, All orders, Archived Orders, End of day forms. All Orders covers every order from the past two years, 30 days per download, and "can take up to 45 minutes"; Amazon gives its XML version only through the API | **Yes**: Orders API v2026-01-01; all-orders and archived-orders reports (30-day request window) | [MODELS] [RTV-ORD] [H-ARCHORD] |
| Manage Returns (FBM) / Return reports (Orders > Manage returns; Site map > Orders > Seller fulfilled return reports) | MFN return requests | Yes | **Yes**: [`GET_FLAT_FILE_RETURNS_DATA_BY_RETURN_DATE`](fbm_returns.md) and related reports | [RTV-RET] |
| FBA Returns page (Site map > Orders > Manage FBA returns) | FBA returns, reasons, ASIN views | "download this information"; also FBA customer returns in Supply chain reports | **Yes**: [`GET_FBA_FULFILLMENT_CUSTOMER_RETURNS_DATA`](fba_returns.md) | [H-FBARET] [RTV-FBA] |
| A-to-z Guarantee claims (Site map > Orders > Manage A-to-z Claims) | Each claim with its status, response deadline and appeal | Not documented | **Partial**: counts and rates come from the V1/V2 performance reports and Data Kiosk account health; money movements from Finances `GuaranteeClaimEventList`. **No API route found for claim-level detail or for responding** | [RTV-PERF] [SA-SCHEMA] [FIN-V0] [H-ATOZ] |
| Chargeback claims (Orders > Manage claims) | Each chargeback | Not documented | **Partial**: counts and rates as above; Finances `ChargebackEventList`. **No API route for claim detail or responses** | [RTV-PERF] [FIN-V0] |
| SAFE-T claims (Site map > Orders > Manage Safe-T claims) | FBM claims for buyer-return issues | Not documented | **Partial**: Finances v0 has `SAFETReimbursementEvent` with `SAFETClaimId`. **No API route found to file a claim.** For seller-fulfilled returns, [fbm_returns](fbm_returns.md) has `SafeT claim id` and `SafeT claim state` columns, so it can show a claim's state (values not yet confirmed); full claim details stay in Seller Central | [FIN-V0] |

### 1.4 Advertising, Stores

| Screen | API route | Sources |
|---|---|---|
| Campaign Manager (Marketing > Ads console) and Sponsored Products/Brands/Display reports | **Yes**: Amazon Ads API (campaign management and reporting); pack page [ads_reports](ads_reports.md) | [ADS] [ADS-ABOUT] |
| Stores (brand owners; Site map > Marketing > Manage stores) | **Yes, per third-party sources**: Ads API `/v2/stores` and store insights | [STORES-API] |

### 1.5 Promotions, coupons, deals, Vine

The live classic menu has these under **Advertising**, not Growth; the new menu has them under **Marketing**.

| Screen | What it shows | UI download | API route | Sources |
|---|---|---|---|---|
| Promotions, Prime exclusive discounts (Marketing > Manage promotions and deals) | Promotions and their performance | Not documented | **Yes (read)**: Promotions API "deals, coupons, price discounts, and basket building promotions"; the documented operations are search and get only. `GET_PROMOTION_PERFORMANCE_REPORT`. Whether Prime exclusive discounts count as "price discounts" is **[unverified]** | [PROMO] [RTV-PERF] |
| Coupons ("Vouchers" in the classic menu) | Coupons; expired or cancelled ones from the last 18 months | Not documented | **Yes**: `GET_COUPON_PERFORMANCE_REPORT`; Promotions API | [RTV-PERF] [PROMO] [H-COUPON] |
| Deals (Lightning/Best Deals) | Deal performance | Not documented | **Yes (read)**: Promotions API covers deals; the Data Kiosk promotion group has `dealId` and related fields | [PROMO] [SA-DK] |
| Vine (brand owners; Marketing > Vine) | Enrolled products, number of Vine reviews | Not documented | **No API route found** | [H-VINE] |

### 1.6 Brands (Brand Registry only; mostly not relevant to a reseller who isn't the brand owner)

| Screen | API route | Sources |
|---|---|---|
| Brand Analytics: Search Terms, SQP, Search Catalog Performance, Market Basket, Repeat Purchase | **Yes**: the `GET_BRAND_ANALYTICS_*` reports, which need Brand Registry and the Brand Analytics role. SQP monthly data goes back at most 17 months | [RTV-AN] |
| Customer reviews / Brand dashboard | **Partial**: Customer Feedback API for review and return topics. **No API route found** for the dashboard itself | [CFA] |
| Manage Your Experiments | **No API route found** | [H-MYE] |

### 1.7 Reports

| Screen | What it shows | UI download | API route | Sources |
|---|---|---|---|---|
| Business Reports, by date and by ASIN (Site map > Other resources > Business reports) | Sales, traffic, Buy Box %, B2B split | CSV; data "generally available for up to two years". Some accounts see a banner on the page telling sellers to switch to Custom Analytics by year-end | **Yes**: [`GET_SALES_AND_TRAFFIC_REPORT`](sales_and_traffic.md) (start no more than 2 years back); Data Kiosk Seller Analytics (start no more than 2 years back); Sales API | [H-BR] [H-BRFAQ] [RTV-AN] [SA-DK] |
| Custom analytics (Site map > Other resources > Custom analytics) | Dashboards of sales, traffic, search, inventory, reviews and more (Amazon says "200+ metrics"): Amazon-curated ones, your own, or ones Amazon's AI builds from a description | Export on each dashboard widget, as an .xlsx file, not CSV ([seller-central-downloads](seller-central-downloads.md)) | Not covered by the research. Its sales and traffic figures overlap [`GET_SALES_AND_TRAFFIC_REPORT`](sales_and_traffic.md) and Data Kiosk [seller_analytics](seller_analytics.md) **[inference]** | Live check |
| Fulfilment reports (Supply chain > Supply chain reports, `/reportcentral/WelcomePage`, titled "Amazon Fulfilment Reports") | All the FBA reports, plus the Amazon VAT Transactions Report and Custom Report Builder | csv or txt. Amazon's note on the page: a csv opened in a spreadsheet drops the leading zeros from MSKUs, ASINs and FNSKUs, so choose txt to keep them. The list may not include a separate long-term storage fee report, Recommended removals or Inventory Age | **Yes**: nearly all have a `GET_…` equivalent | [H-FULREP] [RTV-FBA] |
| Inventory Ledger (summary and detail) | Inventory movements, a "bank statement for your inventory" | csv/txt; 18 months of history. Amazon's banner says it replaced six older reports: Daily Inventory History, Monthly Inventory History, Inventory Event Detail, Inventory Adjustments, Inventory Reconciliation and Received Inventory | **Yes**: [`GET_LEDGER_SUMMARY_VIEW_DATA`](ledger_summary.md), [`GET_LEDGER_DETAIL_VIEW_DATA`](ledger_detail.md) | [H-LEDGER] [RTV-FBA] |
| Custom Report Builder (new) | Drag-and-drop view of inventory data, 30-day range | Download | Same data as the Ledger, so **yes indirectly** | [H-CRB] |
| Payments: Payments Summary / dashboard (Finance > Manage payments) | Total, available, reserved and deferred balance | — | **Yes**: Finances `listBalances`, `listSummary` | [H-PAYSUM] [BAL] [RN] |
| Payments: Statement view (Finance > Finance reports, Statement View tab) | Current statement, unavailable balance (reserve) | — | **Yes**: `listBalances` | [H-STMT] [BAL] |
| Payments: Transaction view (Finance > Finance reports, Transaction View tab) | Transactions; "only transactions from the last 24 months" | Download: a CSV of the transactions in the table, at most 600. The table stops loading at 10,000. For the full list Amazon points to All Statements or the Reports Repository ([seller-central-downloads](seller-central-downloads.md)) | **Yes**: Finances [`listTransactions`](transactions.md) (v2024-06-19; 180-day windows; DEFERRED/RELEASED status) | [H-TXNVIEW] [TXN] |
| Payments: All statements / settlement (Finance > Finance reports, All Statements tab) | Each settlement period | Download Flat File V2, with Flat File and XML in the button's drop-down; generated files kept 90 days | **Yes**: [`GET_V2_SETTLEMENT_REPORT_DATA_FLAT_FILE_V2`](settlements.md). The older flat-file and XML versions are removed on **11 Nov 2026** | [H-ALLSTMT] [RTV-SET] [DEPR] |
| Payments Reports Repository / Date Range reports (Finance > Finance reports, Reports Repository tab) | Transaction report (CSV) or summary report (PDF), up to 365 days per request, back to 2012 | Yes. Report types: Summary, Transaction, Deferred Transaction. The custom date boxes take MM/DD/YYYY. The Transaction file: [date_range_transactions](date_range_transactions.md) | **Partial**: `GET_DATE_RANGE_FINANCIAL_HOLDS_DATA` (Deferred Transaction report) only after you request it in the Repository. The date-range transaction and summary reports themselves have no report type; use the Finances API instead | [H-DRR] [H-REPO] [RTV-PAY] |
| Disbursements / Disbursement tracker (Finance > Finance reports, Disbursements tab) | Payouts and their status | — | **Yes**: Transfers API `listPayouts` and `listExpectedPayouts` (NA/EU/FE). `initiatePayout` is listed for "EU", and UK coverage is **[unverified]** | [H-DISB] [TRANSFERS] |
| Account level reserve | Reserve amount | — | **Yes**: `listBalances` ACCOUNT_LEVEL_RESERVE | [H-ALR] [BAL] |
| Reimbursements | FBA reimbursements, with case IDs | Download, in Supply chain reports | **Yes**: [`GET_FBA_REIMBURSEMENTS_DATA`](reimbursements.md) | [H-PAYREP] [H-REIMB] [RTV-FBA] |
| Advertising reports | Ads console reports | Download. The classic menu's Advertising Reports opens the Ads console, and the Tax Document Library says advertising tax documents are now there too | **Yes**: Ads API reporting ([ads_reports](ads_reports.md)) | [ADS] |
| Tax Document Library (Finance > Finance reports > Tax document library) and Supply chain reports: VAT Calculation report, AVTR, VAT invoice data | VAT data | Yes. The Amazon VAT Transactions Report (AVTR) sits in Supply chain reports, and some accounts see a banner in the Tax Document Library that links to it: csv or txt, one month at a time; last month's report is ready "on or after the third day" of the current month. Amazon's note says it now includes fee data (filter `SALES_CHANNEL` for `AMAZON_FEE` and use `ACTIVITY_PERIOD`). The AVTR file: [vat_transactions](vat_transactions.md) | **Yes**: `SC_VAT_TAX_REPORT` (request only), `GET_VAT_TRANSACTION_DATA` (from the 3rd of each month), `GET_FLAT_FILE_VAT_INVOICE_DATA_REPORT`. Needs a Restricted Data Token | [RTV-TAX] [RTV-INVOICE] |
| Tax Document Library: Amazon VAT invoices (PDF) and seller fee invoices and credit notes | Invoice PDFs; seller fee invoices also as CSV | Yes (7-day request windows for VAT invoices). Seller fee invoices now also come as a transaction-level CSV (transaction ID, order ID, fee type, total fees); the PDFs are unchanged. The page shows the 1,000 most recent invoices, with Load More for older ones ([seller-central-downloads](seller-central-downloads.md)) | **No API route found** for the PDFs or for seller fee invoices | [H-VATINV] [3P-TDL] |
| Selling economics and fees: SKU Economics report (Finance > Profit analytics > Selling economics, `/sereport`) | Sales, units, fees, ad spend, off-Amazon costs and net proceeds per MSKU or another level, for the stores and dates you pick | Generate report, then a CSV; download links stay active for 30 days | **Yes, in a different shape**: Data Kiosk [seller_economics](seller_economics.md) holds the same kind of data as JSON, with different field names | Live check |
| Amazon Selling Coach | Recommendations | — | Not seen in the new menu, the Site map or the classic menu on 28 Oct 2026. Home-page recommendations have no API | — |

### 1.8 Performance

| Screen | What it shows | UI download | API route | Sources |
|---|---|---|---|---|
| Account Health: AHR score and metrics (Account health > Manage account health) | AHR, ODR, late dispatch, cancellations, valid tracking | Its Reports menu has Orders with Defects (cancelled, late, negative-feedback, A-to-z and chargeback orders over the past 12 months, as a weekly chart) and Performance Over Time. No download button was seen on Orders with Defects; whether it ever has one is unconfirmed | **Yes**: [`GET_V2_SELLER_PERFORMANCE_REPORT`](account_health.md) (AHR status and score; counts for each policy category such as authenticity, condition, safety, IP and restricted products) and the Data Kiosk account health group | [RTV-PERF] [SA-SCHEMA] [H-AH] |
| Account Health: individual policy violations by ASIN, with actions and appeals (Product Policy Compliance on the Account Health page; Site map > Account health > Manage compliance) | The actual items with "take action" | Not documented | **Partial**: only category counts come through the V2 report, which has "no ASIN field". `LISTINGS_ITEM_ISSUES_CHANGE` and the Listings Items issues give per-listing enforcement. **No API route found for the full violation list, notice text, uploads or appeal status** | [RTV-PERF] [NOTIF] |
| Account health: regulatory compliance | Compliance problems with at-risk sales, upload docs, Archived tab (6-month view windows) | Not documented | **No API route found** (partly visible through Listings Items issues **[inference]**) | [H-REGCOMP] |
| Performance Notifications (a tab on the Account Health page; Site map > Account health > Performance notifications) | The inbox of Amazon notices from the past 365 days | None seen: the page has a date filter and Mark as read, but no download button | **No API route found.** The only account-level notification is `ACCOUNT_STATUS_CHANGED` (NORMAL / AT_RISK / DEACTIVATED) | [NOTIF] [H-APPROVAL] |
| Feedback Manager (Products > Analyse reviews) | Seller feedback | Neutral and negative only, up to 1-year range, files kept 30 days. The download is the Feedback report, a tab on the Product reports page | **Yes**: [`GET_SELLER_FEEDBACK_DATA`](seller_feedback.md) ("negative and neutral feedback (one to three stars)"). Positive feedback isn't downloadable either way | [H-FEEDBACK] [RTV-PERF] |
| Voice of the Customer (Products > Analyse reviews, Voice of Customer tab; also on the Account Health page) | CX Health, NCX rate, NCX return and review rates, customer comments, root causes | **Download data** button: a CSV summary with one row per listing (NCX rate and orders, star rating, return rate, top NCX reason, CX Health). The first click starts it; click again once it's ready ([seller-central-downloads](seller-central-downloads.md)) | **No API route found** for NCX or CX Health. Partial overlap: Data Kiosk reviews, returns and defect-topic metrics and the Customer Feedback API | [H-VOC] [SA-DK] [CFA] [3P-VOC] |
| Seller University | Training | — | n/a | — |

### 1.9 Messages, cases, approvals, apps

| Screen | What it shows | API route | Sources |
|---|---|---|---|
| Buyer-Seller Messages inbox (Site map > Other resources > Buyer seller messaging) | Incoming buyer messages, Archived folder | **No API route found to read the inbox.** The Messaging API only sends set message types ("You use the Messaging API to send messages to buyers"), and no notification type covers new messages | [MSG] [NOTIF] [H-BSM] [3P-OCTO] |
| Case log, Seller Support (Site map > Support resources > Manage support cases) | Your cases and their correspondence | **Yes (new, read-only)**: Support API `listCases` (filter by status, created/resolved date, subject keywords), `getCase`, `listContacts` (email, phone notes, chat transcripts, attachments). Needs the "Selling Partner Support (Restricted)" role. You can't create or reply | [SUPPORT] [SUPPORT-MODEL] [RN] |
| Approval requests for category or brand ungating (Site map > Products > View selling applications) | Draft and submitted applications and their status | **Partial**: the Listings Restrictions API shows whether an ASIN is restricted for you now, with an "Apply to sell" link. **No API route found for application status or history** | [LR] [H-APPROVAL] [H-SELLAPP] |
| Apps & Services / Appstore (Site map > Settings > Manage your apps; Programs and apps > Selling partner appstore) | Apps you've authorised. Develop Apps, the way into the Solution Provider Portal, is only in the classic menu | **No API route found** | [H-APPSTORE] |
| Seller Assistant (Amazon's own agent) | AI that reads your account and can act and open cases | n/a. It is Amazon's built-in alternative to an outside agent | [H-SELLERASSIST] |

### 1.10 Payments extras, global, settings

| Screen | API route | Sources |
|---|---|---|
| Amazon Lending (Site map > Finance > Lending; some accounts also see a Lending tab in Finance reports). UK lending runs through partners such as YouLend for merchant cash advances | **Partial**: repayments show up as Finances `LoanServicingEventList`. **No API route found for offers, loan terms or balances.** The Data Kiosk "Lending" group is underwriting data for lenders that needs a separate role, not your loan status | [FIN-V0] [SA-SCHEMA] [3P-YOULEND] |
| Seller Wallet (Site map > Finance > Seller Wallet) | **Yes**: Seller Wallet API | [MODELS] |
| Pan-EU, Remote Fulfilment | **Yes**: [`GET_PAN_EU_OFFER_STATUS`](pan_eu_status.md), `GET_MFN_PANEU_OFFER_STATUS`, `GET_REMOTE_FULFILLMENT_ELIGIBILITY` | [RTV-INV] [RTV-FBA] |
| VAT Services on Amazon, the filing service (Site map > Finance > Manage VAT, File VAT) | **No API route found** for the service; the VAT data reports are above | [RTV-TAX] |
| B2B (business pricing, quantity discounts, B2B opportunities) | **Yes**: Listings and Pricing APIs; `GET_B2B_PRODUCT_OPPORTUNITIES_*`; `FEE_DISCOUNTS_REPORT`; `B2B_ANY_OFFER_CHANGED` | [RTV] [NOTIF] |
| Product Opportunity Explorer (Products > Research products) | **Partial**: Customer Feedback API ("same information as… OX" for review and return insights). **No API route found** for niche demand or search volume | [CFA] [H-POE] |
| Settings: account info (Site map > Settings > Account information) | **Partial**: Sellers API [`getAccount`](account.md) (EU only: store participations, business info, selling plan) and [`getMarketplaceParticipations`](store_list.md) | [SELLERS] |
| Settings: user permissions (Site map > Settings > User permissions) | **No API route found** | [H-PERMS] |
| Settings: notification preferences (Site map > Settings > Notification preferences) | **No API route found.** The Notifications API is a separate developer subscription system | [H-NOTIFPREF] [NOTIF] |
| Settings: login and two-step verification (Site map > Settings > Login settings) | n/a | [H-LOGIN] |

---

## 2. The gap list: no API route, grouped by how much it matters to an FBA reseller

The pack's [not-in-the-api](not-in-the-api.md) page lists the screen-only pages with where to find them in Seller Central.

### High
1. **Account health detail.** The list of individual policy violations by ASIN, the notice text, "take action" uploads and appeal status. Only counts and the AHR score come through the API. [RTV-PERF] [SA-SCHEMA]
2. **Performance Notifications inbox.** The API only has `ACCOUNT_STATUS_CHANGED`, and the page has no download. [NOTIF]
3. **Buyer-Seller Messages inbox.** The API can't read it. [MSG] [NOTIF]
4. **Ungating application status and history.** The API shows the current restriction only. [LR] [H-APPROVAL]
5. **FBA capacity limits and Capacity Manager.** [H-CAP] [H-CAPMGR]
6. **IPI score.** [H-IPI]
7. **A-to-z, chargeback and SAFE-T claim detail and responses.** The API gives counts and money, and for seller-fulfilled returns the SAFE-T claim id and state ([fbm_returns](fbm_returns.md)); not the full claim, filing or responses. [RTV-PERF] [FIN-V0]
8. **Settlement history older than about 90 days through the Reports API.** It's a retention limit rather than a missing route: rebuild it through Finances [`listTransactions`](transactions.md) or download it from the UI. [GETREPORTS] [RTV-SET] [TXN]

### Medium
9. **Voice of the Customer.** NCX rate, CX Health and the comments. The page's **Download data** button gives a per-listing CSV. [H-VOC]
10. **Account-health regulatory compliance tab.** Uploads and the Archived tab. [H-REGCOMP]
11. **Seller fee VAT invoices and Amazon VAT invoice PDFs** in the Tax Document Library. Seller fee invoices now also come as a transaction-level CSV there. [H-VATINV] [3P-TDL]
12. **Amazon Lending** offers, terms and balance. [FIN-V0]
13. **Automate Pricing** rules' settings and their history. Rule names and each SKU's rule do come through the API. [H-AUTOPRICE] [AUTOPRICE-API]
14. **Capacity Manager** reservation-fee history (UI download only). [H-CAPMGR]
15. **Ads history beyond the older report types' look-back.** It's a retention limit rather than a missing route: most of the Ads console's Sponsored Ads reports keep 60 to 90 days (gross and invalid traffic 365 days, the budget report 2 years and the Sponsored Products campaign report 5 years), no new ones can be created from 17 Dec 2026, and all of them are deleted after 31 Dec 2026; Ads API v3 keeps 60–95 days for most reports (731 for Sponsored Brands purchased product). So take longer history from unified reporting in the Ads console (24 months of daily or weekly data, 6 years of monthly) or the Ads API's v1 Reporting API, generally available since 30 Sep 2026 ([report-catalogue](report-catalogue.md) §7; [ads_reports](ads_reports.md)). [ADS-SP-PP] [ADS-SP-CAMPAIGN] [ADS-MIGRATE] [ADS-DATA] [ADS-API-RELEASES]
16. **Date Range summary PDF.** The underlying data is in the Finances API. [H-DRR]

### Low (or brand-owner only)
17. User permissions; notification preferences; authorised apps. [H-PERMS] [H-NOTIFPREF]
18. Product Opportunity Explorer niche and search-volume data. [H-POE]
19. Vine, Manage Your Experiments, brand dashboard (brand owners only). [H-VINE] [H-MYE]
20. FBA cost report for EU customs. [H-FBACOST]
21. Seller University and Selling Coach (no account data).

### Closed gaps that used to be UI-only (verified)
- **Case log**: Support API, 30 Sep 2026, read-only, restricted role. [SUPPORT] [RN]
- **Account-level reserve, deferred and available balances**: Finances `listBalances`, 29 Jul 2026. [BAL]
- **Payouts**: Transfers `listPayouts`, 29 Jul 2026. [TRANSFERS] [RN]
- **AHR score and account metrics**: V2 performance report and Data Kiosk Seller Analytics, 30 Sep 2026. [RTV-PERF] [SA-DK]
- **Promotions, deals and coupons (read)**: Promotions API, 26 Aug 2026. [PROMO]
- **Review and return topics**: Customer Feedback API, 25 Jun 2025, UK included, refreshed weekly. [CFA] [RN]
- **Inventory planning, aged stock, storage utilisation**: reports. [RTV-FBA]
- **Capacity limits: not closed.** See Key finding 2.

**Two practical caveats on the new routes:**
- The Support API role is "Restricted". Whether a self-registered private app gets it approved is **[unverified]**.
- The Customer Feedback API needs the Brand Analytics or Selling Partner Insights role. Whether a reseller who isn't the brand owner gets useful ASIN-level data is **[unverified]**.

---

## 3. How far back the UI downloads go (for a 12-month pull)

| Data | UI look-back / limits | API look-back | Sources |
|---|---|---|---|
| Business Reports | "generally available for up to two years"; by-date reports update hourly | Sales & Traffic report and Data Kiosk: start date no more than 2 years ago | [H-BR] [H-BRFAQ] [RTV-AN] [SA-DK] |
| Date Range reports (transaction CSV / summary PDF) | "up to 365 days at a time, going back as far as 2012"; ranges starting 1 Jan 2025 or later use the new format with deferred transactions | Finances [`listTransactions`](transactions.md): 180-day windows; historical limit not stated | [H-DRR] [FORUM-DRR] [TXN] |
| Transaction view | "Only transactions from the last 24 months"; a download holds at most 600 transactions | as above | [H-TXNVIEW] |
| All statements / settlement files | Generated files kept 90 days; older periods can be re-requested ("Request Report"). How far back is **[unverified]** | Reports API keeps reports "for a maximum of 90 days" | [H-ALLSTMT] [GETREPORTS] |
| Inventory Ledger | "up to 18 months of historical inventory data"; "Inventory levels in aggregate are maintained indefinitely" | Detail view: 18 months | [H-LEDGER] [H-FULREP] [RTV-FBA] |
| Amazon Fulfilled Shipments | Up to 18 months back, at most 31 days per request | One month per request | [H-AFS] [RTV-FBA] |
| Manage FBA Inventory report | Shipment data for shipments created in the last 18 months | — | [H-MFI] |
| FBA customer returns, reimbursements, removals | Look-back not stated on the UK help pages **[unverified]** | Not stated | [H-FBARET] [H-REIMB] [H-REMOVAL] |
| Orders | All Orders covers the past 2 years, 30 days per download. Older orders only through Archived Order Reports (monthly, no PII) | 30-day request window; generated reports kept 30 days | [H-ARCHORD] [RTV-ORD] |
| Feedback | Report range up to 1 year; generated files kept 30 days | Not stated | [H-FEEDBACK] |
| Coupons | Expired or cancelled coupons from the last 18 months moved to the new dashboard | — | [H-COUPON] |
| Ads console reports | Unified reporting (Measurement and Reporting > Reporting; generally available 8 Jun 2026, UK included): 24 months of daily or weekly data and 6 years (72 months) of monthly, yearly or summary data; one report covers at most 120 days by day, 15 months by week or 25 months by month. The older Sponsored Ads reports (no new ones from 17 Dec 2026; deleted with their history after 31 Dec 2026): 60 days to 5 years depending on report (purchased product 60 days, search term 65, most others 90, gross and invalid traffic 365 days, budget 2 years, Sponsored Products campaign 5 years; [ads_reports](ads_reports.md) has each) | Ads API v1 Reporting API (generally available since 30 Sep 2026): daily data 24 months, monthly and yearly 6 years. Ads API v3: Sponsored Products 95 days, Sponsored Brands 60 (purchased product 731), Sponsored Display 65 ([report-catalogue](report-catalogue.md) §7) | [ADS-UNIFIED] [ADS-DATA] [ADS-UR-CREATE] [ADS-SP-PP] [ADS-SP-CAMPAIGN] [ADS-MIGRATE] |
| Brand Analytics SQP | — | Monthly: up to 17 months | [RTV-AN] |
| Account health defect snapshots | — | Data Kiosk: 24 months at monthly granularity; returns lag about 45 days | [SA-DK] |
| Regulatory compliance tab | 6-month view windows | — | [H-REGCOMP] |

**For 12 months:** everything except old settlement files is reachable. For ads, use unified reporting in the Ads console or the Ads API's v1 Reporting API (generally available since 30 Sep 2026), not the older report types: most keep only 60 to 95 days, and the console's Sponsored Ads reports can't be created from 17 Dec 2026 and are deleted after 31 Dec 2026. Take settlement data older than 90 days from the UI (All statements → Request Report, or Date Range reports) or rebuild it from the Finances API.

---

## 4. Browser automation: what Amazon's terms say

### 4.1 Which agreement applies to you
A UK seller selling on amazon.co.uk and EU stores signs the **Amazon Services Europe Business Solutions Agreement** (UK help page G201190440). It is governed by Luxembourg law, and its text says "Last updated: June 30, 2025" [BSA-UK]. The UK changes page lists later edits, the latest effective 1 Aug 2026 (a referral-fee clause), and none of them is about agents or AI [BSA-UK-CHANGES]. The US BSA (G1791) applies only to a North America (US/CA) account [BSA-US].

### 4.2 Exact wording

**UK/EU BSA, password security** [BSA-UK]:
> "Any password we provide to you may be used only during the Term to access Seller Central (or other tools we provide) to use the Services… You are solely responsible for maintaining the security of your password. You may not disclose your password to any third party (other than third parties authorized by you to use your account or Seller Central in accordance with this Agreement) and are solely responsible for any use of or action taken under your password."

The UK/EU BSA text retrieved for this research contains no "Agent", "robot", "data mining", "scrap", "automated" access or AI-training clause. The search covered the full text, the changes page and the UK programme-policy changes page [BSA-UK] [BSA-UK-CHANGES] [H-POLCHANGES].

**US BSA (applies to selling in US/CA under the NA agreement)**, section 19, in force since 4 Mar 2026 [BSA-US] [BSA-ANN]:
> "19. Use of Agents. If you deploy, enable, or authorize an Agent to access or interact with any Amazon Services, the following restrictions apply: 19.1 Identification and Compliance Requirements All Agents must clearly identify themselves as automated systems and comply with the requirements specified in the Agent Policy at all times when accessing Amazon Services. Agents are prohibited from accessing Amazon Services if Amazon has explicitly requested cessation of such access. 19.2 Access Limitations Amazon reserves the right, at its sole discretion, to limit or restrict Agent access to Amazon Services through technical or other measures."

> "'Agent' means any software or service that takes autonomous or semi-autonomous action on behalf of, or at the instruction of, any person or entity."

US BSA, licence section [BSA-US]:
> "You will not, and will not allow any third party to, use any of Our Materials or Amazon Services to directly or indirectly develop or improve large language or multimodal models, machine learning models or related technology. You will not and will not allow any third party to use data mining, robots, or similar data gathering and extraction tools to collect or use any of Our Materials or otherwise make any derivative use of any Amazon Service or its contents."

**Agent Policy** (US help page GS83KH2MA7HM69PH, filed under Program Policies › Account Health Rating program policy; the same ID returns nothing on the UK site) [AGENT-US]:
> "Agents must: i. In all HTTP/HTTPS requests, identify that the request is from an Agent and disclose the name of the Agent by including the following in the request's user agent string: "Agent/[agent name]" (e.g., Agent/AmazonAgent) ii. Not conceal or obfuscate that any access, use, or interactions are from an Agent, such as by: (a) mimicking the speed or pattern of human keystrokes, page navigation, or other interactions or (b) completing or circumventing CAPTCHAs or other measures intended to distinguish computers from humans. iii. Respond truthfully to any question or prompt seeking to determine if interactions are coming from a human or a computer. iv. Not circumvent or otherwise avoid any measure intended to block, limit, modify, or control whether and how Agents access, use, or interact with an Amazon Service."

**amazon.co.uk Conditions of Use** (last updated 28 Nov 2025) contain the same "AGENTS" terms and technical rules, applying to "any Amazon Services" [COU-UK]:
> "No Agent may access, use, or interact with Amazon Services unless, at all times, it identifies itself and operates in strict accordance with the requirements in section 3 of these Agent Terms… At our sole discretion, we may limit, including by technical measures, whether and how any Agent accesses, uses, and interacts with Amazon Services."

They also say: "you may not utilise any data mining, robots, or similar data gathering and extraction tools to extract (whether once or many times) for re-utilisation any substantial parts of the content of any Amazon Service, without our express written consent." [COU-UK]

The Conditions of Use govern the shopping site. Whether Amazon treats Seller Central as an "Amazon Service" under them, when the BSA is the seller contract, is **not settled in any source found**.

**SP-API Acceptable Use Policy.** This binds "Solution Providers", meaning anyone, including you, who builds or uses an SP-API app [AUP]:
> "3.3 Do not request or share Amazon Portal usernames or passwords from Authorized Users." / "3.6 Do not ask Authorized Users to share information retrieved from Amazon Portals manually or programmatically to circumvent Amazon policies." / "3.7 If Amazon Portal access is required to provide features or services that benefit Authorized Users, ask the Authorized User to grant access through secondary user permissions." / "4.3 Do not use, offer or promote external (non-Amazon) data services that vend information or data retrieved from Amazon's websites." / "2.4 Be explicit about any calculations and the use of models such as artificial intelligence in the service you provide…" / "2.11 … ensure all submissions and communications are accurate, policy-compliant, and do not result in frivolous or non-compliant contacts or degradation of Amazon support systems."

"Amazon Portal" is defined to include Seller Central [AUP].

**Data Protection Policy** [DPP]:
- "Information" includes anything "exposed through the Amazon Services API, Amazon Portals, or Amazon's public-facing websites", so it covers Seller Central screens.
- It requires MFA, least-privilege access and encrypted credentials.
- It requires PII to be kept "no longer than 30 days after order delivery" except for tax and legal needs.
- In practice: if the agent copies buyer names or addresses out of Seller Central into storage, the DPP retention rules apply.

**UK Seller Central help: user permissions** [H-PERMS]:
> "By granting access, you allow other users to complete tasks… However, account credentials are unique, and confidential information should not be shared with anyone. Remember, you are responsible for the actions of any users that you add to your account…"

Third-party service providers are now authorised through an "authorisation link" generated in the Solution Provider Portal, or through the Service Provider Network. Access is per tool and expires after the service duration you pick.

**Amazon staff on the Seller Forums** (Jessica_Amazon_, about 3 years ago) [FORUM-PERMS]:
> "Don't add external third-party service providers or developers as users." / "Don't share your password with other people."

They point instead to "Authorized Partners".

**Amazon's own agent.** The UK help describes "Seller Assistant – Seller Central's agentic assistant". It is "built on an agentic AI foundation, accessing information across Seller Central", can "take actions on your behalf" and "create and submit support cases". It is switched on per user under User Permissions [H-SELLERASSIST].

### 4.3 What the market does (factual)
- **Reimbursement services.** These have long worked as secondary users with case permissions. Examples:
  - Shopkeeper (page dated 21 Feb 2026) asks for View & Edit on Manage Inventory/Shipments, Orders, Refunds, Returns, Payments and "Manage Your Cases" [3P-SHOPKEEPER].
  - Seller 365 / Threecolts (19 Aug 2026) uses a one-time invitation link and asks for similar roles [3P-SELLER365].
- **New access process.** From 10 Aug 2026 Amazon switched service-provider access to role-based authorisation through the Solution Provider Portal. Providers had to confirm their approved roles by 16 Sep 2026, and existing secondary-user and partner access needs reauthorising **[third-party: beBOLD, AdModifier]** [3P-BEBOLD] [3P-ADMOD]. The UK help page matches the new flow [H-PERMS].
- **Who carries the risk.** Commentators disagree on whether the US Agent Policy reaches sellers' own tools or mainly tool vendors [3P-ECOMSELLER] [3P-ECOMCREW]. No Amazon staff reply answered sellers' questions about GETIDA-type services in the announcement thread [BSA-ANN].
- **Amazon is enforcing against agents in practice.** It sued Perplexity over its Comet shopping agent. A US district court granted an injunction on 9–10 Mar 2026 [3P-GEEKWIRE], and the Ninth Circuit vacated it by 5 Aug 2026 [3P-RIN]. That concerns the shopping site and US computer-access law, not the seller contract.
- **Lock-outs.** The research found no Amazon statement and no reliable report of Seller Central accounts locked specifically for browser automation **[unverified either way]**.

### 4.4 Neutral reading for "an AI agent reading Seller Central in the seller's own logged-in Chrome"
- **Clearly not allowed anywhere:**
  - giving an outside company your password
  - getting round CAPTCHAs or two-step verification
  - copying buyer PII beyond the DPP limits if you are also running an SP-API app
  - using Amazon data to train models (US BSA; and the AUP's AI transparency rule)
- **UK/EU BSA:** no clause on agents or automated reading of your own account. The password clause is about disclosing your password, not about software you run yourself. Risk here is a **grey area**, not an explicit breach.
- **amazon.co.uk Conditions of Use:** have the Agent Terms (identify as an agent, don't mimic humans, no CAPTCHA solving). If Amazon reads "Amazon Services" to include Seller Central, an unlabelled browser agent wouldn't comply.
  - A standard Chrome-extension agent normally uses Chrome's own user-agent string.
  - Whether a given agent tool adds "Agent/…" is **[unverified]**.
- **For an account that covers US/CA:** the BSA section 19 and Agent Policy apply directly, and an unlabelled browser agent wouldn't meet rule (i).
- **Lower-risk routes Amazon explicitly supports:**
  - the SP-API, including the new Support, Finances balances and Data Kiosk routes, for everything in sections 1–2 that has a route
  - Amazon's own Seller Assistant for UI-only questions
  - for a separate human or company: a scoped user permission or service-provider authorisation, never a shared login
- **What stays manual or browser-only:** the gap list in section 2. The terms above still apply.

---

## 5. Assumptions and open points
- This assumes a seller account under the Amazon Services Europe BSA (UK/EU stores), not a unified account that includes North America.
- Settled by the live check on 28 Oct 2026:
  - Voice of the Customer has a UI download: the **Download data** button gives a CSV summary per listing.
  - Account Health has a Reports menu with Orders with Defects and Performance Over Time. No download button was seen on Orders with Defects.
  - Performance Notifications has no download button.
- Not confirmed:
  - how far back settlement re-requests go in the UI
  - the look-back for the FBA returns, reimbursements and removals reports
  - whether Orders with Defects ever offers a download
  - UK coverage of Transfers `initiatePayout`
  - whether Prime exclusive discounts are in the Promotions API
  - whether a private app can get the restricted Support role
- Third-party summaries of the US Agent Policy were checked against Amazon's own US text (quoted above) and match.

---

## Sources

**Amazon SP-API docs** (developer-docs.amazon.com redirects to developer-docs.amazon)
- [MODELS] https://developer-docs.amazon/sp-api/docs/sp-api-models
- [AUTOPRICE-API] https://developer-docs.amazon/sp-api/docs/manage-automated-pricing-rules
- [RN] https://developer-docs.amazon/sp-api/docs/sp-api-release-notes
- [DEPR] https://developer-docs.amazon/sp-api/docs/sp-api-deprecations
- [RTV] https://developer-docs.amazon/sp-api/docs/report-type-values
- [RTV-FBA] https://developer-docs.amazon/sp-api/docs/report-type-values-fba
- [RTV-INV] https://developer-docs.amazon/sp-api/docs/report-type-values-inventory
- [RTV-ORD] https://developer-docs.amazon/sp-api/docs/report-type-values-order
- [RTV-RET] https://developer-docs.amazon.com/sp-api/docs/report-type-values-returns
- [RTV-PERF] https://developer-docs.amazon/sp-api/docs/report-type-values-performance
- [RTV-PAY] https://developer-docs.amazon/sp-api/docs/report-type-values-payment
- [RTV-SET] https://developer-docs.amazon/sp-api/docs/report-type-values-settlement
- [RTV-AN] https://developer-docs.amazon/sp-api/docs/report-type-values-analytics
- [RTV-TAX] https://developer-docs.amazon/sp-api/docs/report-type-values-tax
- [RTV-INVOICE] https://developer-docs.amazon.com/sp-api/docs/report-type-values-invoice-data
- [NOTIF] https://developer-docs.amazon/sp-api/docs/notification-type-values
- [SUPPORT] https://developer-docs.amazon/sp-api/docs/support-api
- [SUPPORT-MODEL] https://raw.githubusercontent.com/amzn/selling-partner-api-models/main/models/support-api-model/support_2025-02-01.json
- [PROMO] https://developer-docs.amazon/sp-api/docs/promotions-api
- [CFA] https://developer-docs.amazon/sp-api/docs/customer-feedback-api
- [DK] https://developer-docs.amazon/sp-api/docs/data-kiosk-api
- [SA-DK] https://developer-docs.amazon/sp-api/docs/seller-analytics-dataset-guide
- [SA-SCHEMA] https://github.com/amzn/selling-partner-api-models/blob/main/schemas/data-kiosk/analytics_sellerAnalytics_2025_03_31.graphql
- [BAL] https://developer-docs.amazon/sp-api/reference/listbalances
- [TXN] https://developer-docs.amazon/sp-api/reference/listtransactions
- [GETREPORTS] https://developer-docs.amazon/sp-api/reference/getreports
- [TRANSFERS] https://developer-docs.amazon/sp-api/docs/transfers-api
- [SELLERS] https://developer-docs.amazon/sp-api/docs/sellers-api
- [MSG] https://developer-docs.amazon/sp-api/docs/messaging-api
- [LR] https://developer-docs.amazon/sp-api/docs/listings-restrictions-api
- [FIN-V0] https://raw.githubusercontent.com/amzn/selling-partner-api-models/main/models/finances-api-model/financesV0.json
- [FEEDS-FBA] https://developer-docs.amazon.com/sp-api/docs/fba-feeds-attributes (feed name from a search snippet)
- [GH-5179] https://github.com/amzn/selling-partner-api-models/discussions/5179

**Amazon Ads**
- [ADS-API-RELEASES] https://advertising.amazon.com/API/docs/en-us/release-notes/index
- [ADS] https://advertising.amazon.com/API
- [ADS-ABOUT] https://advertising.amazon.com/about-api
- [ADS-UNIFIED] https://advertising.amazon.com/resources/whats-new/streamline-campaign-analysis-with-unified-reporting (8 Jun 2026)
- [ADS-DATA] https://advertising.amazon.com/help/G8A5Z6UD9ME5W3GZ (Data availability in unified reporting)
- [ADS-UR] https://advertising.amazon.com/help/GMH8A8AJSH4ATV6T (Understand unified reporting)
- [ADS-UR-CREATE] https://advertising.amazon.com/help/GK3FG568T93VE2VL (Create a report in unified reporting)
- [ADS-MIGRATE] https://advertising.amazon.com/help/GK3JZZE5QLTF8U8A (Migrate your scheduled reports)
- [ADS-SP-PP] https://advertising.amazon.com/help/G9JR57B7AEWBQ3P2 (Sponsored Products purchased product report)
- [ADS-SP-CAMPAIGN] https://advertising.amazon.com/help/G4KJE82E7LDKAHZX (Sponsored Products campaign report)

**Amazon agreements and policies**
- [AUP] https://sellercentral.amazon.com/solution-provider/policy?policyType=AUP&locale=en_US
- [DPP] https://sellercentral.amazon.com/solution-provider/policy?policyType=DPP&locale=en_US
- [BSA-UK] https://sellercentral.amazon.co.uk/help/hub/reference/external/G201190440
- [BSA-UK-CHANGES] https://sellercentral.amazon.co.uk/help/hub/reference/external/G201476470
- [H-POLCHANGES] https://sellercentral.amazon.co.uk/help/hub/reference/external/GQHQGBTD7XB7EECN
- [BSA-US] https://sellercentral.amazon.com/help/hub/reference/external/G1791
- [AGENT-US] https://sellercentral.amazon.com/help/hub/reference/external/GS83KH2MA7HM69PH
- [BSA-ANN] https://sellercentral.amazon.com/seller-forums/discussions/t/84e3f6b1-42f7-4cf3-a189-a5cc8d78d838
- [COU-UK] https://www.amazon.co.uk/gp/help/customer/display.html?nodeId=201909000
- [COU-US] https://www.amazon.com/gp/help/customer/display.html?nodeId=GLSBYFE9MGKKQXXM

**UK Seller Central help** (https://sellercentral.amazon.co.uk/help/hub/reference/external/<ID>)

*Inventory and capacity*
- [H-CAP] GAFNWEYTJUV2GBFC (FBA capacity limits)
- [H-CAPMGR] GM6SJGQDA625AUCJ (Capacity Manager)
- [H-IPI] GZJF4DY2W6MERBAL (IPI FAQ)
- [H-INVPERF] G202174810 (Inventory performance)
- [H-FBAINV] GTMXYZN64UJL7TT6 (FBA Inventory overview)
- [H-FBAINVRPT] GKT9YKCHXMSDVJKB (FBA Inventory report)
- [H-MFI] G200740930 (Manage FBA Inventory report)
- [H-MAI] G41 (Manage All Inventory)
- [H-RESTOCK] G201634550 (Restock Inventory)
- [H-STRANDED] G201835130 (Stranded inventory report)
- [H-REMOVAL] G200989090 (Removal reports), G200989110 (Removal Order Detail)
- [H-FBACOST] G5Q7F55FDWTX4LJZ (FBA cost report)

*Reports*
- [H-FULREP] G201104750 (Amazon Fulfilment reports)
- [H-LEDGER] G4FKT5KQWFFJ7LDN (Inventory Ledger)
- [H-CRB] GEQ8JEWH5KJC5QCG (Custom Report Builder)
- [H-AFS] G200453120 (Amazon-Fulfilled Shipments)
- [H-FBARET] G202191620 (FBA Returns page), G200453320 (FBA customer returns report)
- [H-REIMB] G200732720 (Reimbursements report)
- [H-BR] G202142060 (types of Business reports)
- [H-BRFAQ] G28761 (Business Reports FAQ)
- [H-ARCHORD] GQE5U7B4QEZY5HEJ (Archived Order reports)

*Payments and tax*
- [H-PAYREP] G200885400 (Payments reports)
- [H-PAYSUM] G200913190 (Payments Summary)
- [H-STMT] G200378770 (Statement View)
- [H-TXNVIEW] G200378780 (Transaction View)
- [H-ALLSTMT] G200801160 (View All Statements)
- [H-DRR] G200989190 (Date Range reports)
- [H-REPO] GTQ4DK8DQKMHBQJ9 (Payments Reports Repository)
- [H-DISB] GFLRKM5TG3LHNCV8 (Disbursement Tracker)
- [H-ALR] G200136810 (account level reserve)
- [H-VATINV] GH3ZSPRW2VTSD57S (Download Amazon VAT invoices)

*Performance, pricing and growth*
- [H-FEEPREV] G201115050 (Fee Preview Report)
- [H-AUTOPRICE] G201994820 (Automate Pricing)
- [H-COUPON] GT3QSSD2KV9QV74Q (new coupon dashboard)
- [H-VINE] GQGXHUH63WG79ELA (Track Vine reviews)
- [H-MYE] GVP453K5XRBJS7Y9 (Manage Your Experiments)
- [H-POE] GNJ4YRTXWLMBY38U (Product Opportunity Explorer)
- [H-AH] GX663LDD3TGU86GX (Monitor your account health)
- [H-REGCOMP] G2T95N7ZAX4CVYMN (Account health – regulatory compliance)
- [H-VOC] GRTPNYUAU5EQ9RGU (Voice of the Customer)
- [H-FEEDBACK] G202125660 (Download feedback report)
- [H-ATOZ] G27951 (About A-to-z claims)

*Messages, approvals, settings and apps*
- [H-BSM] G200383320 (Buyer-Seller Messages FAQ), GLLCKBC8JFQLGUGB (Search and filter messages)
- [H-APPROVAL] G200333160 (Categories and products that require approval)
- [H-SELLAPP] G84EFS4CKK43USMF (Selling Applications FAQ)
- [H-PERMS] G901 (Set and edit user permissions)
- [H-NOTIFPREF] G871 (Notification preferences)
- [H-LOGIN] G831 (Change login settings)
- [H-APPSTORE] G201687890 (Selling Partner Appstore)
- [H-SELLERASSIST] GLYJTRGQCYNYAZS9 (Seller Assistant)

**Amazon Seller Forums (staff posts)**
- [FORUM-PERMS] https://sellercentral-europe.amazon.com/seller-forums/discussions/t/d575bdfe-df5d-4b0d-9bd8-6d4ce66cfe8e
- [FORUM-DRR] https://sellercentral-europe.amazon.com/seller-forums/discussions/t/5a7d44ff-358e-4c39-a6de-e67aff1a2770
- [FORUM-CAP] https://sellercentral.amazon.com/seller-forums/discussions/t/3c9cef3c-530b-4846-a3b3-a855ed62f241

**Third-party (secondary)**
- [STORES-API] https://ppc.land/amazon-brand-stores-api-exits-beta-what-brands-can-now-automate/
- [3P-VOC] https://www.sellerlabs.com/blog/amazon-voice-of-the-customer-transition-2026/
- [3P-TDL] https://linkmybooks.com/blog/amazon-seller-tax-documents
- [3P-OCTO] https://docs.octocom.ai/docs/amazon
- [3P-YOULEND] https://ibsintelligence.com/ibsi-news/amazon-and-youlend-launch-flexible-financing-program-for-uk-smbs
- [3P-SHOPKEEPER] https://shopkeeper.com/guide/en/guide-for-setting-user-permissions
- [3P-SELLER365] https://intercom.help/seller-365/en/articles/10514319-how-to-provide-seller-central-access-for-the-seller-365-reimbursements-service
- [3P-BEBOLD] https://www.bebolddigital.com/news/amazon-seller-central-service-provider-authorization
- [3P-ADMOD] https://admodifier.com/amazon-simplified-service-provider-authorization-365-day-expiry/
- [3P-ECOMSELLER] https://ecomsellertool.com/blog/amazon-agent-policy-ai-tools
- [3P-ECOMCREW] https://www.ecomcrew.com/amazon-bans-ai-agents-seller-platform/
- [3P-GEEKWIRE] https://www.geekwire.com/2026/03/10/judge-blocks-perplexitys-ai-bot-from-shopping-on-amazon-in-early-test-of-agentic-commerce/
- [3P-RIN] https://www.retail-insight-network.com/news/us-court-overturns-amazon-injunction-perplexity/
