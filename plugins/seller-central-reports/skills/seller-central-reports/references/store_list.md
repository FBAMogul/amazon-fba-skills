# Marketplace participations (`store_list`)

Amazon's list of the stores "where the seller can list items" and "the seller's participation in those marketplaces": each store's details, whether the seller participates there, whether the seller's listings there are set to inactive, and the store name the seller trades under ([getMarketplaceParticipations](https://developer-docs.amazon/sp-api/reference/getmarketplaceparticipations)).

## Where to find it in Seller Central

Not confirmed as one page. Per-store listing status, which `hasSuspendedListings` reflects, is set in the holiday settings; Amazon's help gives **Settings > Account Settings > Store Status > Account Overview > Holiday Settings** ([Listing status for vacations, holidays, and other absences](https://sellercentral.amazon.co.uk/help/hub/reference/external/G200135620)). The new main menu has no Settings heading: the **Site map** (at the bottom of the main menu) lists **Settings**, with **Account information** (`/hz/sc/account-information`) and **Account settings** (`/account/management`). Checked in Seller Central UK on 28 Oct 2026. The store status setting itself wasn't opened.

## Getting it through the API

- **Call / report type:** Sellers API v1 `getMarketplaceParticipations`. It is a direct API call, not a report, and takes no parameters ([model](https://github.com/amzn/selling-partner-api-models/blob/main/models/sellers-api-model/sellers.json)). The response body is the data.
- **Role:** in the EU, Product Listing or Selling Partner Insights. The other listed roles apply in the Far East region only ([role mappings](https://developer-docs.amazon/sp-api/docs/role-mappings)).
- **How far back Amazon goes:** not stated in Amazon's docs. The call takes no dates and returns the account as it is now.

## Fields

Checked against a real UK seller's file (Oct 2026): every field path. The file is JSON.

| Field | Meaning | Format and traps |
|---|---|---|
| `payload[]` | One entry per store. | Can include "Non-Amazon" entries as well as Amazon stores (see Traps). |
| `payload[].marketplace.id` | Amazon's id for the store. | For example `A1F83G8C2ARO7P` for the UK ([Store Identifiers](https://developer-docs.amazon/sp-api/docs/store-identifiers)). |
| `payload[].marketplace.name` | Store name. | For example `Amazon.fr`; `Non-Amazon <country>` for the Non-Amazon entries. |
| `payload[].marketplace.countryCode` | Country of the store. | Two letters. The UK is `GB`, not `UK`. |
| `payload[].marketplace.defaultCurrencyCode` | The store's currency. | Three-letter currency code, such as `GBP` or `EUR`. |
| `payload[].marketplace.defaultLanguageCode` | The store's language. | A locale such as `fr_FR`, not the bare ISO 639-1 code the model says. |
| `payload[].marketplace.domainName` | The store's domain name. | `www.amazon.…` for Amazon stores (for example `www.amazon.fr`). The Non-Amazon entries have `si-prod…` or `siprod…` subdomains of `stores.amazon.*`. |
| `payload[].participation.isParticipating` | `true` if the seller participates in the store, otherwise `false`. | Not the same as selling there (see Traps). |
| `payload[].participation.hasSuspendedListings` | `true` if the seller's Listing Status is set to Inactive, otherwise `false`. | Not reliable: can disagree with the same field in [`account`](account.md) on the same day. |
| `payload[].storeName` | The seller's store name as shown in that store. | Can differ from store to store. |

Documented but not in the file: `errors[]` (only in a failed response).

Source: [Sellers API model (sellers.json)](https://github.com/amzn/selling-partner-api-models/blob/main/models/sellers-api-model/sellers.json) and [Get the market participation of a seller](https://developer-docs.amazon/sp-api/docs/get-seller-market-participation).

## Personal data

`storeName` is the seller's trading name, a business detail. No buyer data. Strip it before analysis and never copy it into anything shared.

## Questions it answers

- Which stores does Amazon count the seller as participating in (`isParticipating`), and has that changed since an earlier copy? It can't say whether listings are live in a store: `hasSuspendedListings` isn't reliable (see below).
- Does the seller have a store name set in each store? (Have the stripping script add a yes or no column for `storeName`, then drop it; the name itself is the seller's own, shown in Seller Central.)
- What currency and language does each store use? This helps read per-store figures in the other reports.

## Traps seen in real data

Seen in a real UK seller's file (Oct 2026):

- **More entries than Amazon stores.** Alongside the Amazon stores, the file can list "Non-Amazon" entries on `si-prod…` or `siprod…` domains. Keep only entries whose `domainName` starts with `www.amazon.` when you want the Amazon stores. Orders from a Non-Amazon entry can still show in [`orders`](orders.md) and [`transactions`](transactions.md) as Multi-Channel Fulfilment.
- **Participating is not selling.** An account can show as participating in a store where it has no live listings or sales. Check [`listings_all`](listings_all.md) or [`orders`](orders.md) for that.
- **`hasSuspendedListings` disagrees with [`account`](account.md):** for the same store on the same day, the two files can give different values. Don't rely on it.
- **Store names can vary by store,** and can differ from the name the seller uses elsewhere.
