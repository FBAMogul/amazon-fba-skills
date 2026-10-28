# Pan-European Eligibility: FBA ASINs (`pan_eu_status`)

Amazon's "Pan-European Eligibility: FBA ASINs" report: Pan-European FBA enrolment status and eligibility for each Amazon-fulfilled listing, with its offer status in each European store ([SP-API inventory reports](https://developer-docs.amazon/sp-api/docs/report-type-values-inventory)).

## Where to find it in Seller Central

**Supply chain > Supply chain reports** (the Amazon Fulfilment Reports page), then **Inventory > Pan-European Eligibility: FBA ASINs** in the left-hand list (`/reportcentral/panEUInventoryReport/1`; **Show more...** expands the Inventory list). **Pan-European Eligibility: Self-fulfilled ASINs**, next to it, is a separate report. Classic: **Reports > Fulfilment by Amazon**. Checked in Seller Central UK on 28 Oct 2026.

Amazon's help says the **Pan-European FBA Inventory** page shows each ASIN's status and offers the same download ([Pan-European FBA product eligibility](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201855620)), and links the page as `/paneu/inventory` ([G201855610](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201855610)). Classic Seller Central has it as **Inventory > Manage Pan-EU FBA Inventory**. The new menu and its site map had no entry for it on 28 Oct 2026.

## Getting it through the API

- **Call / report type:** Reports API 2021-06-30 `createReport` with `reportType` `GET_PAN_EU_OFFER_STATUS` and no `dataStartTime`/`dataEndTime`; then `getReport` and `getReportDocument`. Amazon makes it available in the Spain, UK, France, Germany and Italy stores only ([SP-API inventory reports](https://developer-docs.amazon/sp-api/docs/report-type-values-inventory)). Amazon doesn't say whether this report wants one marketplace id or several, or what it does with ids for stores where it isn't available. Asked once with every store's marketplace id (Oct 2026), it returned a single file.
- **Report options:** none.
- **Role:** any one of Amazon Fulfillment, Inventory and Order Tracking, Pricing, Product Listing, or Direct to Consumer Shipping (restricted) ([role mappings](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types)).
- **How far back Amazon goes:** not stated in Amazon's docs. Each file is the status as it stood when asked for, so there is no back history.

## Columns

Checked against a real UK seller's file (Oct 2026): 30 columns, in this order. The file is tab-separated text with a header row first. It starts with a UTF-8 byte order mark. The headers contain spaces, so quote them when reading.

Amazon defines none of these columns; meanings are read from the name and the values.

| Column | Meaning | Format and traps |
|---|---|---|
| `ASIN` | Amazon's product identifier. | |
| `MerchantSKU` | Your SKU. | Unique per row. Can include old SKUs, among them Amazon-made SKUs in the `XX-XXXX-XXXX` form. |
| `Enrol` | Whether the SKU is enrolled in Pan-EU. | `N` when not enrolled. Amazon doesn't document the value for an enrolled SKU. |
| `FNSKU` | Amazon's fulfilment network SKU. | |
| `Pan-EU status` | Pan-EU eligibility. | `Eligible`, `Ineligible` or `Failed to retrieve data`. Not the Pan-EU page's wording. |
| `Enrollment Date` | When the SKU was enrolled. | Blank on SKUs that aren't enrolled. |
| `Title` | Product title. | |
| `UK offer status` | The SKU's offer in the UK store. | `No listing`, or the UK price as text with a space after `£` (`£ <price>`) when there's an offer. |
| `DE offer status`, `FR offer status`, `IT offer status`, `ES offer status`, `NL offer status`, `SE offer status`, `PL offer status` | Whether the SKU has, needs or can't have an offer in that store. | `No listing`, `No offer required`, or what blocks an offer, e.g. `Product type`; several blockers are joined with a comma and a space. |
| `BE Offer Status`, `IE Offer Status` | The same for Belgium and Ireland. | Capital O and S. IE has blockers of its own, e.g. `Size`. |
| `Last active on` | Not defined. | Can be blank. |
| `Date Pan-EU expires` | Not defined. | Can be blank. |
| `Product comments` | Not defined. | Can be blank. |
| `UK PanEU benefits` … `PL PanEU benefits`, `BE PanEU Benefits`, `IE PanEU Benefits` | Whether the SKU gets Pan-EU benefits in that store. | `N` when it doesn't. BE and IE use a capital B. |

Documented but not in the file: `UK SnL Status`, `DE SnL Status`, `FR SnL Status`, `IT SnL Status`, `ES SnL Status`.

Source: [SP-API inventory reports](https://developer-docs.amazon/sp-api/docs/report-type-values-inventory) (column names only).

## Personal data

None. No buyer or business personal details.

## Questions it answers

- Which ASINs are enrolled in Pan-EU, which are eligible but not enrolled, and which are ineligible?
- Which enrolled ASINs are missing an active offer in a required store? Enrolment needs an active offer in amazon.de, .fr, .it and .es, plus amazon.nl from 3 September 2026 and amazon.com.be from 26 February 2027, unless an exemption applies; UK stock is handled separately ([Pan-European FBA ASIN enrolment](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201855610)). The UK left Pan-European FBA in 2021.
- Which ASINs are about to lose Pan-EU benefits (`Date Pan-EU expires`), so an offer must be fixed first?
- Which ASINs have no offer in an optional store (SE, PL, IE) that could be listed?

## Traps seen in real data

Seen in a real UK seller's file (Oct 2026):

- **More rows than current stock.** The file can list old SKUs, including Amazon-made `XX-XXXX-XXXX` ones, so it can hold more SKUs than [`inventory_all`](inventory_all.md).
- **`Eligible` doesn't mean enrolled.** A SKU can be `Eligible` with `Enrol` `N` and every benefits column `N`.
- **`Failed to retrieve data` rows** have only `ASIN`, `MerchantSKU`, `FNSKU` and `Pan-EU status`; every other column is blank.
- **`UK offer status` holds a price** (`£ <price>`) for SKUs with a UK offer, and `No listing` otherwise.
- **No SnL (Small and Light) columns,** though Amazon lists five.
- Starts with a UTF-8 byte order mark; lines end in CRLF, but the last line has no line ending at all, so `wc -l` counts one row short and a shell `while read` loop skips that row.
