# Inbound plans (`inbound_plans`)

A list of the account's FBA inbound plans (Send to Amazon), one short summary per plan: name, status, stores, dates and ship-from address; `listInboundPlans` "provides a list of inbound plans with minimal information" ([listInboundPlans](https://developer-docs.amazon/sp-api/reference/listinboundplans)).

## Where to find it in Seller Central

**Supply chain > Manage shipments** (`/manage/shipments`) for the shipments list, and **Supply chain > Send to Amazon** (`/fba/sendtoamazon`) to start a plan. Classic: **Inventory > Shipments**, in its FBA group (`/gp/ssof/shipping-queue.html`, the Shipping Queue). Checked in Seller Central UK on 28 Oct 2026. Amazon's help says the Shipping Queue lists the shipments the seller is working on and those sent to Amazon ([Track your shipments: Shipping queue and shipment summary](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201022330)), and Send to Amazon starts from there ([Create shipments with Send to Amazon](https://sellercentral.amazon.co.uk/help/hub/reference/external/G6925SDD66GDLXJW)).

## Getting it through the API

- **Call / report type:** not a report. Fulfillment Inbound API 2024-03-20 `listInboundPlans`, with `pageSize` up to 30 (Amazon's maximum), paging with `paginationToken` taken from `pagination.nextToken`. The response's `inboundPlans` holds one object per plan ([listInboundPlans](https://developer-docs.amazon/sp-api/reference/listinboundplans)).
  - **Ask once per status.** Amazon's docs don't say what leaving out `status` returns; in real use it returned `ACTIVE` plans only. To list every plan, call it with `status` `ACTIVE`, `SHIPPED` and `VOIDED` in turn and join the rows; if a plan changes status between calls and is listed twice, keep its later row. `ERRORED` isn't a filter value, so errored plans can't be listed.
  - **Drafts may be missing.** Plans made in Send to Amazon reach the API only after their placement and transportation options are confirmed ([Fulfillment Inbound API](https://developer-docs.amazon/sp-api/docs/fulfillment-inbound-api)).
  - **Summary only.** Amazon's FAQ says this call returns shipment, packing, placement and transportation ids, then points to the other GET calls for detail ([Fulfillment Inbound API FAQ](https://developer-docs.amazon/sp-api/docs/fulfillment-inbound-faq)); but the plan summary in Amazon's model has none of those ids, so expect only the fields below. Use `getInboundPlan` and the other GET calls for detail.
- **Role:** [Amazon Fulfillment](https://developer-docs.amazon/sp-api/docs/role-mappings) only.
- **How far back Amazon goes:** not stated in Amazon's docs. It returns the plans as they stand now, not a date window. One guide says sellers can filter by creation or last-update date, but the operation's model has only sort options for those (`sortBy`, `sortOrder`), no date filter ([Additional Functionality](https://developer-docs.amazon/sp-api/docs/additional-functionality-fulfillment-inbound)). Old plans still come back (see Traps seen in real data).

## Columns

JSON: the response's `inboundPlans` list holds one object per plan, with the same seven top-level fields on every plan.

| Column | Meaning | Format and traps |
|---|---|---|
| `inboundPlanId` | Id of the inbound plan | `wf` followed by 36 characters. Every id starts `wf`. |
| `name` | Human-readable name of the plan | Can be an empty string; otherwise whatever the seller typed. |
| `status` | Current status: `ACTIVE` (being worked on), `SHIPPED` (completed; only minor changes possible), `VOIDED` (all shipments cancelled), `ERRORED` | `ACTIVE` doesn't mean recent (see Traps seen in real data). |
| `marketplaceIds[]` | Store ids the plan is for | An array, e.g. `["A1F83G8C2ARO7P"]` for the UK store. One store per plan seen. |
| `createdAt` | When the plan was created | ISO 8601, UTC (`YYYY-MM-DDThh:mm:ssZ`). |
| `lastUpdatedAt` | When the plan was last updated | Same format. |
| `sourceAddress.name` | Primary contact at the ship-from address | Personal. |
| `sourceAddress.companyName` | Business name | Personal. Not on every plan. |
| `sourceAddress.addressLine1` | Street address | Personal. |
| `sourceAddress.addressLine2` | More street address | Personal. Not on every plan. |
| `sourceAddress.city` | City | Personal. |
| `sourceAddress.stateOrProvinceCode` | State or province code | Personal. Can be a county name rather than a code. |
| `sourceAddress.postalCode` | Postal code | Personal. |
| `sourceAddress.countryCode` | Country, ISO 3166-1 alpha-2 | e.g. `GB`. |
| `sourceAddress.phoneNumber` | Phone number | Personal. On every plan. |
| `sourceAddress.email` | Email address | Personal. Not on every plan. |

Documented but not in the response: `sourceAddress.districtOrCounty`.

## Personal data

Every `sourceAddress` field except `countryCode`: the ship-from contact name, company name, street address, city, county, postcode, phone number and email. Strip `sourceAddress` from every plan before analysis and never copy any of it into anything shared. A saved API response holds the plans inside `inboundPlans`, so use `jq 'del(.inboundPlans[]?.sourceAddress)'` on it; `jq -c 'del(.sourceAddress)'` works only on a file that holds one plan per line, and on a saved response removes nothing.

## Questions it answers

- Which inbound plans are still active, and which have shipped or been voided?
- When was each plan created and last touched, and which stores is it for?
- Has an old active plan been left open that should be finished or cancelled?
- Which plan ids to look up in Seller Central (or with `getInboundPlan`) for shipment and item detail?

## Traps seen in real data

Seen in a real UK seller's API responses (Oct 2026):

- **Without `status`, only `ACTIVE` plans come back.** With `status` set, Amazon honours it together with `paginationToken`, and asking for `ACTIVE`, `SHIPPED` and `VOIDED` in turn listed every plan once, each `inboundPlanId` unique. Plans in `ERRORED` status can't be listed.
- **`ACTIVE` doesn't mean recent.** Old plans can stay `ACTIVE` long after their stock was shipped. Amazon's docs don't say when a plan moves from `ACTIVE` to `SHIPPED`, so filter by `createdAt` or `lastUpdatedAt` as well as `status`, and don't read an old `ACTIVE` plan as stock that was never sent; check its shipments in Seller Central.
- **`name` can be an empty string.**
- **`stateOrProvinceCode` can be a county name**, not a code.
- **The ship-from address is personal data:** a contact name, phone number, email and street address. Never copy any of it.
- **No shipment, packing, placement or transport ids:** just the summary fields above.

Source: `InboundPlanSummary` and `Address` in [fulfillmentInbound_2024-03-20.json](https://github.com/amzn/selling-partner-api-models/blob/main/models/fulfillment-inbound-api-model/fulfillmentInbound_2024-03-20.json); status meanings from [listInboundPlans](https://developer-docs.amazon/sp-api/reference/listinboundplans).
