---
name: Retrieve, reshop and cancel a Breeze order
description: Service an existing Breeze Airways order — retrieve it, quote a change, remove individual flights or cancel the whole order with OrderReshop, then commit with OrderChange.
api: Breeze Airways NDC Gateway
docs: https://ndc.flybreeze.com/docs/ndc-for-developers/ndc-servicing/ndc-servicing-flows
generated: '2026-08-01'
method: generated
operations:
  - POST /api/Servicing/r3.x/v21.3/OrderRetrieve
  - POST /api/Servicing/r3.x/v21.3/OrderQuote
  - POST /api/Servicing/r3.x/v21.3/OrderReshop
  - POST /api/Servicing/r3.x/v21.3/OrderChange
---

# Retrieve, reshop and cancel a Breeze order

All Servicing routes require `Authorization: Bearer <session token>` and
`Ocp-Apim-Subscription-Key`. Get a **fresh** token for the servicing flow — a token is a
reservation-system session and must not be shared with a concurrent booking flow.

## Step 1 — Retrieve the order

`POST /api/Servicing/r3.x/v21.3/OrderRetrieve` with an `IATA_OrderRetrieveRQ`. The
response is an `IATA_OrderViewRS` carrying the order, its `OrderItem` list, passengers,
and — after a cancellation has been processed — any travel credit amount that was issued.

Use this as your source of truth before and after every change. It is also the correct
way to reconcile after a timeout, because the gateway has no idempotency contract.

## Step 2 — Quote the change (optional)

`POST /api/Servicing/r3.x/v21.3/OrderQuote` with an `IATA_OrderQuoteRQ` returns an
`IATA_OrderReshopRS` describing the price impact of a proposed change.

## Step 3 — Reshop

`POST /api/Servicing/r3.x/v21.3/OrderReshop` with an `IATA_OrderReshopRQ`. No `OfferID`
is used on the request; the response carries its own reshop offer identifier
(e.g. `id-RUNUNTNS-o-3`).

Two shapes:

- **Cancel the whole order (void)** — use the `CancelOrderRef` node. `OrderRefID` is
  mandatory in the request payload.
- **Remove individual items** — use the `DeleteOrderItem` node, referencing the
  `OrderItemRefID` of the specific flight segment or service to remove.

**Known limitation.** The current implementation cannot apply penalties or spoilage
disclosed in fare rules during an OrderReshop session. Do not present a reshop response
as the final refund figure to a customer.

## Step 4 — Commit with OrderChange

`POST /api/Servicing/r3.x/v21.3/OrderChange` with an `IATA_OrderChangeRQ` commits the
reshop selection. The response is an `IATA_OrderViewRS`.

Refunds and travel credit are **asynchronous**: once the seller completes the change, an
automated worker processes any refund or travel credit. Poll
`POST /api/Servicing/r3.x/v21.3/OrderRetrieve` — a credit amount appears on that response
when it has been issued. Do not tell the customer a refund amount before it appears there.

## Refund eligibility rules

No refund or credit is processed unless one of these holds:

- the fare is refundable (BreezeCorp `BY` and Federal Government `FG` fares are
  refundable; `EZ` Flex and `NO` No Flex are not), **or**
- the cancellation occurs within 24 hours of the initial booking **and** the flight is
  scheduled to depart at least 168 hours from the time of cancellation.

## Errors

Servicing failures surface as NDC `Errors/Error` elements (`TypeCode`, `DescText`,
`LangCode` `en`) and as `Error-Code-0` / `Error-Msg-0` response headers. See
`errors/breeze-airways-error-codes.yml`. A `Connect ETIMEDOUT` means your IP is not
allowlisted; a `401 Access Denied` means the subscription key is missing or wrong for
this environment.
