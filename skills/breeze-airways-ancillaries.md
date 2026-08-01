---
name: Add bags, bundles and seats to a Breeze offer
description: Use ServiceList and SeatAvailability against a priced Breeze offer to attach checked bags, carry-on, bundles and seat assignments before creating the order.
api: Breeze Airways NDC Gateway
docs: https://ndc.flybreeze.com/docs/ndc-for-developers/ndc-shopping/ServiceList
generated: '2026-08-01'
method: generated
operations:
  - POST /api/Selling/r3.x/v21.3/ServiceList
  - POST /api/Selling/r3.x/v21.3/SeatAvailability
  - POST /api/Selling/r3.x/v21.3/OrderCreate
---

# Add bags, bundles and seats to a Breeze offer

Ancillaries are sold against a **priced** offer. Run
`POST /api/Selling/r3.x/v21.3/OfferPrice` first and carry its `OfferID` (`sid-…`) into
both routes below. All Selling routes require `Authorization: Bearer <session token>`
plus `Ocp-Apim-Subscription-Key`.

## Step 1 — List services

`POST /api/Selling/r3.x/v21.3/ServiceList` with an `IATA_ServiceListRQ` referencing the
priced offer. The response (`IATA_ServiceListRS`) carries the ancillaries your agency is
authorized to sell.

Breeze service codes:

| Code | Service |
|---|---|
| `CB1` | Checked bag 1 |
| `CB2` | Checked bag 2 |
| `CB3` | Checked bag 3 |
| `COB` | Carry-on bag (a single SSR) |
| `BZU` | Nicer bundle (OTA / leisure partners only) |
| `AZU` | Nicest bundle (OTA / leisure partners only) |

**Checked bag SSRs are cumulative and must be attached in sequence**: `CB1`, then `CB2`,
then `CB3`. Do not attach `CB2` without `CB1`. Carry-on is a single, non-cumulative SSR.

Bundles are not available to TMC or specialized TMC (federal government) partners.
`AS2003` means bundle information is not available at the shopping stage.

## Step 2 — Get the seat map

`POST /api/Selling/r3.x/v21.3/SeatAvailability` with an `IATA_SeatAvailabilityRQ`
referencing the priced offer `sid-…`. The response returns a new response identifier
(`id-…`) — use it only where the response tells you to; do not send it back as the offer
reference.

Seats carry `SeatCharacteristicCode` values from IATA PADIS code set 9825. A historical
defect (issue `DISTRO-92`, resolved 2025-10-12) returned code `1` — *Restricted seat -
General* — for standard economy seats. Validate your seat display against the PADIS
reference published at
`https://ndc.flybreeze.com/docs/ndc-for-developers/ndc-shopping/SeatAvailability/IATA-PADIS-seat-codes`.

Breeze offers both `$0` seats and paid seats. A Nicer bundle sample must include a
selected `$0` seat; an additional Nicer sample should include an upgraded seat.

## Step 3 — Commit the selections

Selected services and seats are carried into
`POST /api/Selling/r3.x/v21.3/OrderCreate` as additional `SelectedOfferItem` entries
under `AcceptSelectedQuotedOfferList/SelectedPricedOffer`, each with its own
`OfferItemRefID` and the `PaxRefID` values it applies to.

## Adding ancillaries after booking

For an order that already exists, use the Servicing twins instead:
`POST /api/Servicing/r3.x/v21.3/ServiceList` and
`POST /api/Servicing/r3.x/v21.3/SeatAvailability`. Never use the Selling `OrderCreate`
route against an existing order.

## Caution

There is no idempotency key on this gateway. Re-sending a service selection can attach a
duplicate paid ancillary. Confirm state with
`POST /api/Servicing/r3.x/v21.3/OrderRetrieve` before retrying anything that costs money.
