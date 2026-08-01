---
name: Shop Breeze flights and create an order
description: Authenticate against the Breeze Airways NDC gateway, search markets for offers, price a selected offer, and commit it into an order.
api: Breeze Airways NDC Gateway
docs: https://ndc.flybreeze.com/docs/category/ndc-for-developers
generated: '2026-08-01'
method: generated
operations:
  - POST /api/Selling/r3.x/Auth
  - POST /api/Shopping/r3.x/v21.3/AirShopping
  - POST /api/Selling/r3.x/v21.3/OfferPrice
  - POST /api/Selling/r3.x/v21.3/OrderCreate
---

# Shop Breeze flights and create an order

Breeze Airways distributes through an IATA NDC 21.3 gateway. Every payload is XML in
the `http://www.iata.org/IATA/2015/EASD/00/IATA_OffersAndOrdersMessage` namespace and
must declare `PayloadAttributes/VersionNumber` = `21.3` and a `DistributionChain` link
with `OrgRole` `Seller` and your agency `OrgID`.

## Before you start

- Your calling IP must be on the Breeze allowlist for the environment you are hitting.
  A non-allowlisted call fails with `Connect ETIMEDOUT`.
- Send `Ocp-Apim-Subscription-Key` on **every** request. It is unique per partner and
  per environment. A wrong value returns `401 Access Denied` with
  `WWW-Authenticate: AzureApiManagementKey`.
- Base hosts: sandbox `https://ndcr4y.test.mx.navitaire.com`, production
  `https://ndcr4y.prod.mx.navitaire.com`.

## Step 1 — Get a session token

`POST /api/Selling/r3.x/Auth` with `Authorization: Basic <base64 credentials>` and body
`{"grant_type":"client_credentials"}`. Add `?role={rolecode}` when Breeze has issued you
one — the request is about twice as fast because the gateway skips an extra
reservation-system lookup.

The response is `{"access_token": "<JWT>", "token_type": "Bearer", "expires_in": "00:30:00"}`.

**This token is a stateful session, not a stateless credential.** Get a new token for
each transaction flow. Reusing one token concurrently across bookings will fail.

If you get `500` with `Error-Code-0: CF4000`, read `Error-Msg-0`:
`Unable to find best role for agent…` means the `role` parameter does not match the
credential (fix or drop it); `Unable to decode credentials…` means the Authorization
header is missing or malformed; `The agent … is locked` means you must contact Breeze.

## Step 2 — Search for offers

`POST /api/Shopping/r3.x/v21.3/AirShopping` with an `IATA_AirShoppingRQ`. This route
requires **no** Authorization header, but still requires the subscription key.

Build `Request/FlightRequest/FlightRequestOriginDestinationsCriteria` with one
`OriginDestCriteria` per leg (`OriginDepCriteria` with `Date` and `IATA_LocationCode`,
`DestArrivalCriteria` with `IATA_LocationCode`), a `PaxList` of `Pax` entries with
`PaxID` and `PTC` (`ADT`, `CHD`, `INFT`), and `ResponseParameters/CurParameter/CurCode`.

Rules the gateway enforces:

- Infants may not outnumber adults (`AS4003`/`AS4004`/`AS4005`), and each infant must be
  associated to a distinct adult (`AS4047`).
- Do not allow child-only reservations — Breeze does not carry unaccompanied minors.

The response returns many `OfferID` values, often several per flight for different fare
products. Match `JourneyPriceClass/PriceClassRefID` against `PriceClassList` to tell
Flex (`EZ`) from No Flex (`NO`). `AS2000` means no offers; `AS2001` means the market or
date is unavailable; `AS2003` means bundle information is not available in shopping.
Breeze operates less-than-daily service, so an empty result is normal.

## Step 3 — Price the offer

`POST /api/Selling/r3.x/v21.3/OfferPrice` with `Authorization: Bearer <token>` and an
`IATA_OfferPriceRQ` carrying one or many `OfferID`s **from the same offer**. Pricing
across two different offers errors.

The response returns a **single, new** priced `OfferID` (`sid-…`). Discard the shopping
`OfferID` — identifiers are re-minted at each step and reusing a stale one can undo the
change you intended.

## Step 4 — Create the order

`POST /api/Selling/r3.x/v21.3/OrderCreate` with `Authorization: Bearer <token>` and an
`IATA_OrderCreateRQ`. Use this route only for new orders; modifying an existing order
goes through the Servicing routes.

Populate `Request/CreateOrder/AcceptSelectedQuotedOfferList/SelectedPricedOffer` with the
priced `OfferRefID`, `OwnerCode` `NV`, and one `SelectedOfferItem` per `OfferItemRefID`
with its `PaxRefID` list. Populate `DataLists/ContactInfoList` and the passenger list.

Breeze validates the data it must hand to authorities and use for disruption contact:

- SecureFlight data — name, gender, date of birth — is required at time of booking.
- The reservation contact email must be the passenger's own. Agency proxy or relay
  addresses are rejected.
- Phone is preferred for operational SMS and must include the country code.

The response is an `IATA_OrderViewRS` carrying the created order.

## Retry and safety

Breeze publishes **no idempotency key and no safe-retry contract**. Do not blind-retry
`OrderCreate`. If a call times out, obtain a fresh token and use
`POST /api/Servicing/r3.x/v21.3/OrderRetrieve` to determine whether the order exists
before attempting anything again.
