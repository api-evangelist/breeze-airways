# Breeze Airways

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
