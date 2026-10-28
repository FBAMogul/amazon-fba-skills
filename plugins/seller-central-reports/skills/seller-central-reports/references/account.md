# Seller account details (`account`)

Amazon's record of the seller account and its stores: business type, selling plan, registered business details, primary contact and store participations ([getAccount](https://developer-docs.amazon/sp-api/reference/getaccount)).

**This file is personal and business data throughout.** Read the Personal data section below before opening it.

## Where to find it in Seller Central

**Settings > Account information** (`/hz/sc/account-information`), in the Business Information section ([Change your business address](https://sellercentral.amazon.co.uk/help/hub/reference/external/G201975730), [Update account information](https://sellercentral.amazon.co.uk/help/hub/reference/external/G191)). Amazon's help still calls the page Account Info. The new main menu has no Settings heading: the **Site map** (at the bottom of the main menu) lists it. Checked in Seller Central UK on 28 Oct 2026. The Business Information section itself wasn't opened.

## Getting it through the API

- **Call / report type:** Sellers API v1 `getAccount`. It is a direct API call, not a report, and takes no parameters ([model](https://github.com/amzn/selling-partner-api-models/blob/main/models/sellers-api-model/sellers.json)). The response body is the data. The call "is only available in the Amazon EU store" ([Get the account information of a seller](https://developer-docs.amazon/sp-api/docs/get-account-information-eu-seller)).
- **Role:** Finance and Accounting or Account Information Service Provider, EU only ([role mappings](https://developer-docs.amazon/sp-api/docs/role-mappings)).
- **How far back Amazon goes:** not stated in Amazon's docs. The call takes no dates and returns the account as it is now.

## Fields

Checked against a real UK seller's file (Oct 2026): every field path. The file is JSON. Address objects (`registeredBusinessAddress`, `primaryContact.address`) have the same fields, described once as `<address>.…`.

| Field | Meaning | Format and traps |
|---|---|---|
| `payload.businessType` | Type of business registered for the account. | One of `CHARITY`, `CRAFTSMAN`, `NATURAL_PERSON_COMPANY`, `PUBLIC_LISTED`, `PRIVATE_LIMITED`, `SOLE_PROPRIETORSHIP`, `STATE_OWNED` or `INDIVIDUAL` ([model](https://github.com/amzn/selling-partner-api-models/blob/main/models/sellers-api-model/sellers.json)). |
| `payload.sellingPlan` | Selling plan. | `PROFESSIONAL` or `INDIVIDUAL` ([model](https://github.com/amzn/selling-partner-api-models/blob/main/models/sellers-api-model/sellers.json)). |
| `payload.marketplaceParticipationList[]` | The stores, each with `marketplace`, `participation` and `storeName`. | Same shape and stores as [`store_list`](store_list.md), including any "Non-Amazon" entries, but `hasSuspendedListings` can disagree with `store_list` on the same day. |
| `payload.business.name` | Registered business name. | Personal data. |
| `payload.business.nonLatinName` | Registered business name in non-Latin script, if applicable. | Can be an empty string. |
| `payload.business.registeredBusinessAddress` | Registered business address (address fields below). | Personal data. |
| `payload.business.companyRegistrationNumber` | Company registration number, if applicable. | Personal data. |
| `payload.business.companyTaxIdentificationNumber` | Company tax identification number, if applicable. | Personal data. |
| `payload.primaryContact.name` | Full name of the primary contact. | Personal data. |
| `payload.primaryContact.nonLatinName` | Primary contact's name in non-Latin script, if applicable. | Can be an empty string. |
| `payload.primaryContact.address` | The primary contact's residential address (address fields below). | Personal data: a home address. |
| `<address>.addressLine1`, `addressLine2`, `city`, `stateOrProvinceCode`, `postalCode`, `countryCode` | The address. | All six keys are present in both addresses; `stateOrProvinceCode` can be an empty string. |

Documented but not in the file: `errors[]` (only in a failed response).

Source: [Sellers API model (sellers.json)](https://github.com/amzn/selling-partner-api-models/blob/main/models/sellers-api-model/sellers.json).

## Personal data

Almost every field: `business.name`, `business.nonLatinName`, `business.registeredBusinessAddress`, `business.companyRegistrationNumber`, `business.companyTaxIdentificationNumber`, `primaryContact.name`, `primaryContact.nonLatinName`, `primaryContact.address` (the contact's home address), and each store's `storeName`. Only `businessType` and `sellingPlan` are safe to show.

Read only the field you need. Never copy a value from this file anywhere, don't quote it back unless the seller asks, and never put it into anything shared. To see whether something changed, compare copies without printing the values.

## Questions it answers

- Is the account on the Professional selling plan?
- What business type is the account registered as with Amazon?
- Has Amazon's record of the registered business details or primary contact changed since an earlier copy? Compare copies without printing the values.
- Which stores does Amazon list against the account?

## Traps seen in real data

Seen in a real UK seller's file (Oct 2026):

- **Every documented field can be there as a key with nothing in it.** A field can come as an empty string rather than being left out, so a file can hold a key with no value. Check for empty strings, not just missing keys.
- **The store list matches [`store_list`](store_list.md)** (Amazon stores plus any "Non-Amazon" entries), but `hasSuspendedListings` can differ between the two files for the same store on the same day. Don't rely on either.
