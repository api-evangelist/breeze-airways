# Breeze Airways

Breeze Airways (Breeze Aviation Group, Inc.) is an American low-cost, point-to-point airline headquartered in Cottonwood Heights, Utah, founded by David Neeleman and flying since May 2021.

Its programmable surface is a partner-facing **IATA NDC gateway** — IATA Offers and Orders 21.3 XML messages over a Navitaire-hosted, IP-allowlisted gateway — documented at the Breeze B2B portal. Access is limited to accredited travel agency partners (OTA, TMC, and US federal government) with an IATA/ARC number and an executed commercial agreement. Breeze also distributes over Amadeus (1A) and Travelport+ (1G) GDS and through the TravelPro agent booking portal.

## API

**Breeze Airways NDC Gateway** — IATA NDC 21.3
- Docs: https://ndc.flybreeze.com/docs/category/ndc-for-developers
- Production: `https://ndcr4y.prod.mx.navitaire.com`
- Sandbox: `https://ndcr4y.test.mx.navitaire.com`

Thirteen documented routes across three families:

| Family | Routes |
|---|---|
| Selling (auth) | `Auth` |
| Shopping | `AirlineProfile`, `AirShopping` |
| Selling | `OfferPrice`, `ServiceList`, `SeatAvailability`, `OrderCreate` |
| Servicing | `OrderRetrieve`, `OrderQuote`, `OrderReshop`, `OrderChange`, `ServiceList`, `SeatAvailability` |

Every request carries an `Ocp-Apim-Subscription-Key`; all routes except `AirShopping` also require a 30-minute session bearer token from `POST /api/Selling/r3.x/Auth`.

## What Breeze publishes — and does not

| | |
|---|---|
| OpenAPI / Swagger | No — `/openapi.json`, `/openapi.yaml`, `/swagger.json` all miss |
| AsyncAPI / webhooks | No — though issued tokens carry an `orderchangenotification` audience |
| SDKs / packages | No first-party client libraries on npm, PyPI or any registry |
| CLI, UI components | No |
| MCP server | No (a *candidate* tool set is derived in `mcp/`, not published by Breeze) |
| A2A agent card | No — `/.well-known/agent-card.json` and `/.well-known/agent.json` both miss on every host |
| `/.well-known/*` | Nothing served on any host |
| security.txt / VDP / trust center | None found |
| Idempotency | Not documented — no key, no safe-retry contract |
| Rate limits | Not published; volume governed commercially at go-live |
| Status page | The B2B news feed is labelled "News & system status" |

## Artifacts

| Dir | File |
|---|---|
| `authentication/` | Subscription key, Basic, session bearer, client-credentials, per-route table |
| `scopes/` | JWT token audiences and role codes (no requestable OAuth scopes) |
| `errors/` | NDC gateway error codes, error envelope, transport failures, known issues |
| `conventions/` | NDC message semantics, identifier re-minting, sessions, business rules |
| `conformance/` | IATA NDC 21.3, ONE Order, PADIS 9825, OAuth2, JWT, SecureFlight, EDIFACT |
| `lifecycle/` | Versioning, compatibility statement, status feed, certification and go-live |
| `changelog/` | Recent dated entries from the B2B news feed |
| `sandbox/` | Test/production environments, access prerequisites, sample markets, test matrix |
| `data-model/` | Offer / OfferItem / Order / OrderItem / Pax entity graph |
| `skills/` | Three agent skills: shop-and-book, ancillaries, servicing |
| `mcp/` | Candidate MCP tool set (derived, not published by Breeze) |
| `well-known/` | Probe results for every `/.well-known/` path on every host |
| `security/` | Domain security probe (TLS, HSTS, DNSSEC, CAA, SPF, DMARC) |
| `llms/` | Generated `llms.txt` |

Enriched by the API Evangelist pipeline, 2026-08-01.
