# Getting your own SP-API access

How a UK seller gets Selling Partner API (SP-API) access to their own account, keeps it working and keeps the keys safe. Amazon calls this a **private developer** with a **private application** that you **self-authorise** ([Amazon](https://developer-docs.amazon/sp-api/docs/register-as-a-private-developer), [Amazon](https://developer-docs.amazon/sp-api/docs/self-authorization)). Researched 27 Oct 2026; Amazon changes these steps, so check its pages if a screen doesn't match.

Walk the seller through it one step at a time. Never ask them to paste a key, secret or token into the chat.

## Before you start

- **A Professional selling plan.** "Individual accounts are not eligible" ([Amazon](https://developer-docs.amazon/sp-api/docs/sp-api-registration-overview)).
- **The primary user** of the Seller Central account (the owner login, not an invited user). A non-primary user who tries to authorise gets error SPDC8143 ([Amazon](https://developer-docs.amazon/sp-api/docs/resolve-common-http-and-authorization-error-codes)).
- **Documents for the identity check:** a government photo ID (passport, national ID card or driving licence); proof of address (a bank, credit card or payment-provider statement from the last 180 days); business details such as the company registration number and registered address ([Amazon](https://developer-docs.amazon/sp-api/docs/solution-provider-identity-verification), [Amazon](https://developer-docs.amazon/sp-api/docs/onboarding-step-1-prepare-for-registration)).

## Step by step

1. **Open the Solution Provider Portal.** Go straight to solutionproviderportal.amazon.com, or go through Seller Central. The new Seller Central has no Develop Apps menu entry, so choose **Classic Seller Central** at the foot of the ☰ menu, then **Apps and Services → Develop Apps**, which sends you on to the portal (checked in Seller Central UK on 28 Oct 2026). Either way the portal asks you to sign in again: use the existing Seller Central login rather than a new account. "Developer Central" was the old name ([Amazon](https://developer.amazonservices.com/register), [FAQ](https://developer-docs.amazon/sp-api/docs/spp-faq)).
2. **Verify your identity.** Choose **Get started**, then: the business information form, a one-time PIN to your phone, the primary contact form, upload ID and proof of address, then the separate identity check in a new tab. Details can't be edited after you submit. Amazon says it "usually takes 20 minutes"; the review can take "up to two business days" and the result comes by email ([Amazon](https://developer-docs.amazon/sp-api/docs/solution-provider-identity-verification)).
3. **Fill in the Developer Profile** ([Amazon](https://developer-docs.amazon/sp-api/docs/onboarding-step-3-create-a-developer-profile)):
   - **Data access:** choose "Private Developer: I build application(s) that integrate my own company with Amazon Services APIs".
   - **Roles:** see the table below.
   - **Use cases:** plain words, e.g. "Internal dashboard for our own Amazon UK account: sales, stock levels, fees and payouts. Used only by the business owner." Answer truthfully about customer data. Even without the restricted roles, `orders`, `fba_shipments` and `fba_shipment_sales` carry the buyer's city, county and postcode, and the Amazon Fulfilled Shipments report (`fba_shipments`) also has a buyer email column. So don't write "No customer personal data" if the app will fetch those reports: say what it gets and how you handle it (for example, the buyer columns are stripped as soon as a file arrives).
   - **Security controls:** the person responsible for security answers. Stay under 500 words, write original content, and don't copy from Amazon's policies ([Amazon](https://developer-docs.amazon/sp-api/docs/sp-api-registration-overview)). Answers should match what you actually do.
   - **Agreements:** accept the Solution Provider Portal Agreement, Acceptable Use Policy (AUP) and Data Protection Policy (DPP), then **Register**.
4. **Wait for approval.** Amazon opens a case and emails you; if it asks a question, reply **within five days** or the case closes with no access ([Amazon](https://developer-docs.amazon/sp-api/docs/register-as-a-private-developer)). Amazon says standard roles are "typically taking one to two weeks" ([Amazon](https://developer-docs.amazon/sp-api/docs/onboarding-step-3-create-a-developer-profile)), and the portal FAQ says "Plan for 5–10 business days" ([FAQ](https://developer-docs.amazon/sp-api/docs/spp-faq)). Some developers report waiting several weeks.
5. **Create the app.** Choose **Add new app client**, give it a name, choose **API type: SP-API** and tick the roles (only approved ones can be ticked). Private apps don't need OAuth login or redirect URLs. **Save and exit**: the app stays in **Draft**, and that's fine, as "You can self-authorize your application in draft status" ([Amazon](https://developer-docs.amazon/sp-api/docs/onboarding-step-7-register-your-first-production-application), [Amazon](https://developer-docs.amazon/sp-api/docs/self-authorization)).
6. **Get the LWA client ID and client secret** from the **View** link in the app list's **LWA credentials** column ([Amazon](https://developer-docs.amazon/sp-api/docs/rotating-your-apps-lwa-credentials)).
7. **Self-authorise for a refresh token.** Open the app's **Authorize app** page and click **Authorize app**. Each click makes a new refresh token and earlier ones stay valid. There's a **maximum of 10 self-authorisations per private app**, so don't click it to test ([Amazon](https://developer-docs.amazon/sp-api/docs/application-authorization-limits)). Amazon's error guide says refresh tokens last 365 days, but its re-authorisation rule is written for public apps; whether a private app's token expires is unverified ([Amazon](https://developer-docs.amazon/sp-api/docs/renew-authorizations)).
8. **UK settings.** Endpoint: Europe, `https://sellingpartnerapi-eu.amazon.com` (AWS region `eu-west-1`), which covers the UK and the EU stores ([Amazon](https://developer-docs.amazon/sp-api/docs/sp-api-endpoints)). UK marketplace ID: `A1F83G8C2ARO7P` ([Amazon](https://developer-docs.amazon/sp-api/docs/marketplace-ids)). No AWS account is needed: since 2 Oct 2023 the SP-API needs no IAM role or request signing ([Amazon](https://developer-docs.amazon/sp-api/changelog/sp-api-will-no-longer-require-aws-iam-or-aws-signature-version-4)), so skip old tutorials that ask for one. Libraries swap the refresh token for a one-hour access token for you ([Amazon](https://developer-docs.amazon/sp-api/docs/connecting-to-the-selling-partner-api)).
9. **Sandbox (optional).** `https://sandbox.sellingpartnerapi-eu.amazon.com` returns fake responses only, never the seller's real data, and allows 5 requests a second (burst 15). Use it to check code runs, not to look at figures ([Amazon](https://developer-docs.amazon/sp-api/docs/sp-api-sandbox)).

## Which roles to tick

Tick only what you'll use: the AUP says "Do not request access to or retrieve Information that is not necessary for your Application's functionality" (§3.8). The first six roles fetch every report page in this pack except two ([role mappings](https://developer-docs.amazon/sp-api/docs/role-mappings-for-types), [roles](https://developer-docs.amazon/sp-api/docs/roles-in-the-selling-partner-api)): `vat_transactions` needs Tax Invoicing, a restricted role, plus a Restricted Data Token, so most sellers download it in Seller Central ([vat_transactions](vat_transactions.md)); and `date_range_transactions` has no API route, so download it in Seller Central ([date_range_transactions](date_range_transactions.md)).

| Role | What it gets in this pack |
|---|---|
| Inventory and Order Tracking | `orders`, `fba_shipments`, `fbm_returns`, `listings_all`, `pan_eu_status` |
| Amazon Fulfillment | FBA stock, ledger, reimbursements, returns, removals, storage fees, fee preview, inbound plans, inbound problems (`inbound_noncompliance`), `fba_shipment_sales`; also `orders`, `fba_shipments`, `seller_feedback`, `pan_eu_status` |
| Finance and Accounting | `transactions`, `settlements`, `account` |
| Selling Partner Insights | `account_health`, `account_health_v1`, `seller_feedback`, `store_list`; also `seller_economics`, with Brand Analytics |
| Product Listing | `listings_all`, `suppressed_listings`, `pan_eu_status`, `store_list`; also gating checks (Listings Restrictions) |
| Brand Analytics | `sales_and_traffic` and every Data Kiosk query (`seller_economics`, which also needs Selling Partner Insights, `seller_analytics`, `seller_analytics_asin`) |
| Pricing (optional) | Live offers and Buy Box prices. No report in this pack needs it. |

- **Brand Analytics:** Amazon's roles page doesn't say it needs Brand Registry, but whether every reseller is granted it isn't confirmed. The Brand Analytics search-term reports still need Brand Registry.
- **Leave the restricted roles unticked:** Direct-to-Consumer Shipping, Tax Invoicing, Tax Remittance, Professional Services and Selling Partner Support. They unlock buyer names and addresses or support cases, and add a three-stage review: business, security, then a call with an Amazon solution architect ([Amazon](https://developer-docs.amazon/sp-api/docs/onboarding-step-3-create-a-developer-profile)). A seller's dashboard needs none of them; order reports still give the buyer's postcode without them.

## Upkeep

- **Renew the client secret every 180 days.** Amazon notifies you 90 days before. Only the primary user can do it: use the alert in the **LWA credentials** column and choose **Rotate secret**. The old secret stops working 7 days after the new one is made. Miss the deadline and every call returns an error ([Amazon](https://developer-docs.amazon/sp-api/docs/rotating-your-apps-lwa-credentials)). The seller then updates `.env` themselves.
- **Make a successful call at least every 90 days.** "Access keys that do not make a successful call in 90 days will be deleted and the Solution Provider will need to re-apply" ([AUP](https://sellercentral.amazon.com/mws/static/policy?documentType=AUP&locale=en_GB) §3.5).
- **Self-authorise again after adding a role**, or the token won't carry it ([Amazon](https://developer-docs.amazon/sp-api/docs/roles-in-the-selling-partner-api)). That uses one of the 10.

## Keeping the keys safe

- The three values (client ID, client secret, refresh token) go in a `.env` file in the project folder. The seller types them in themselves in their editor, never in the chat.
- List `.env` in the project's `.gitignore`, so it's never committed.
- Block Claude from reading it with a deny rule in the project's `.claude/settings.json` ([Claude Code docs](https://code.claude.com/docs/en/settings)):

```json
{
  "permissions": {
    "deny": ["Read(./.env)", "Read(./.env.*)"]
  }
}
```

  That rule covers Claude's own file tools and commands that name the file, such as `cat .env`. It doesn't cover a command that searches a whole folder (such as `grep -r`) or a script that opens files itself ([Claude Code docs](https://code.claude.com/docs/en/permissions)). So the ground rule still applies: Claude never opens, prints or searches `.env`, and scripts read the keys only to make their API calls. For a lock that holds for every command, turn on Claude Code's sandbox ([sandboxing](https://code.claude.com/docs/en/sandboxing)).

- "Never share access keys or passwords" ([AUP](https://sellercentral.amazon.com/mws/static/policy?documentType=AUP&locale=en_GB) §3.1): not in a GitHub repo, a screenshot, a Discord message or a VA's spreadsheet. If one leaks, do these in order:
  1. **Revoke the self-authorisation.** Self-authorising again isn't enough, because "Generating a new refresh token doesn't invalidate previous refresh tokens" ([Amazon](https://developer-docs.amazon/sp-api/docs/self-authorization)). In the Solution Provider Portal, open the drop-down next to the app's **Edit** and choose **Authorize** to reach Manage Authorizations; under **Revoke Authorization**, expand **Self-Authorizations** and "Choose Revoke" for the authorisation ([Amazon](https://developer-docs.amazon/sp-api/docs/revoke-authorizations)). If you can't tell which one the leaked token came from, revoke them all.
  2. **Rotate the client secret** (see Upkeep). The old secret keeps working for up to 7 days after the new one is made ([Amazon](https://developer-docs.amazon/sp-api/docs/rotating-your-apps-lwa-credentials)). If that window is a problem, contact Developer Support: "You can also open a case to invalidate your old LWA credentials before the seven-day period" ([FAQ](https://developer-docs.amazon/sp-api/docs/lwa-credentials-faq)).
  3. **Self-authorise again** for a new refresh token (it uses one of the 10), and update `.env` yourself.
  4. **Tell Amazon within 24 hours** ([Agreement](https://sellercentral.amazon.com/mws/static/agreement?locale=en_GB) D-6.3).

## Data Protection Policy basics for a one-person seller

The [DPP](https://sellercentral.amazon.com/mws/static/policy?documentType=DPP&locale=en_GB) covers "all systems that store, process or otherwise handle data" from the API, so it covers the seller's own laptop and scripts. For one person it mostly means ([DPP](https://sellercentral.amazon.com/mws/static/policy?documentType=DPP&locale=en_GB), [Aug 2026 update](https://developer.amazonservices.com/policy-update-for-sp-api)):

- MFA on accounts that reach the data, and passwords of 12+ characters.
- Screen lock after at most 15 minutes; disk encryption on (FileVault on a Mac, BitLocker on Windows); anti-malware with automatic updates.
- API credentials encrypted at rest, rotated at least every 12 months, never hardcoded in code, never in a public repository.
- Separate test and production set-ups.
- Report a security incident to security@amazon.com within 24 hours. Amazon may audit you; keep records for 12 months.
- Anything reachable from the internet needs a web application firewall "or equivalent controls", so keep dashboards **local** (a spreadsheet or an app on the seller's own computer).
- Keep data only "for so long as … strictly necessary", and delete it within 30 days when Amazon asks.
- Customer personal data is for "selected tax and merchant fulfilled shipping purposes" only, and must be deleted within 30 days after delivery. The DPP's "Information" includes data from "Amazon Portals", not only the API, so this applies to Seller Central downloads too. Before Claude reads the rows of a report with buyer columns, it runs a small script on your computer that copies the file without them, and it works only from that copy (see the ground rules in [SKILL.md](../SKILL.md)). Never paste raw buyer data into a chat.
- **AI:** the [Agreement](https://sellercentral.amazon.com/mws/static/agreement?locale=en_GB) §6 bans using Amazon's materials, data included, to develop or improve AI models. No wording found bans analysing your own data with an AI. On Claude Free, Pro or Max, turn off the model-training setting at claude.ai/settings/data-privacy-controls ([Anthropic](https://www.anthropic.com/news/updates-to-our-consumer-terms)).

Breaking the AUP or DPP "may result in suspension or termination of Amazon Services API access".

## Common errors

| Error | Cause and fix |
|---|---|
| 403 "Unauthorized" | The role wasn't approved, or was added without self-authorising again; wrong region (UK must use `sellingpartnerapi-eu`, not `-na`); expired or rotated client secret; the seller account is deactivated; an old API version Amazon has removed ([Amazon](https://developer-docs.amazon/sp-api/docs/resolve-common-http-and-authorization-error-codes)). |
| 403 on Data Kiosk | Every Data Kiosk call needs the Brand Analytics role ([Amazon staff reply](https://github.com/amzn/selling-partner-api-models/issues/5325)). Seller Economics queries also need Selling Partner Insights ([Amazon](https://developer-docs.amazon/sp-api/changelog/correction-data-kiosk-role-requirements)). |
| "Marketplace not valid for region" | A marketplace ID from another region on the EU endpoint. Use `A1F83G8C2ARO7P` for the UK ([GitHub](https://github.com/amzn/selling-partner-api-models/issues/5126)). |
| CONSENT_LIMIT_REACHED | All 10 self-authorisations are used ([Amazon](https://developer-docs.amazon/sp-api/docs/application-authorization-limits)). |
| SPDC8143 | Signed in as a secondary user. Sign in as the primary user. |
| "Individual accounts are not eligible" | Upgrade to the Professional plan first. |
| 429 Too Many Requests | Throttled. Wait and retry with growing gaps; use reports for bulk history ([Amazon](https://developer-docs.amazon/sp-api/docs/usage-plans-and-rate-limits)). |
| Report status CANCELLED | No data for the period, or a repeat request inside the report's limit. Not an error. |
| Calls fail after about six months | The client secret hit its 180-day expiry. Rotate it. |
| A tutorial asks for an AWS IAM ARN or access key | Not needed since Oct 2023. Skip it. |
