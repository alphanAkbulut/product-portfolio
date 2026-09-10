# Airline Ancillary Services

## Multi-Provider Retailing & Checkout

An interactive product case study exploring how heterogeneous airline ancillary content can be transformed into a consistent, passenger- and segment-aware B2B checkout experience.

> Portfolio reconstruction based on professional product-management experience. All company names, internal services, endpoints, identifiers, credentials, production data, and provider-specific contracts have been removed or replaced with generic equivalents.

![Airline ancillary checkout prototype](../../Ekran%20Resmi%202026-09-10%2011.04.55.png)

## Explore the prototype

[View the preserved HTML demo](../../ancillaries_and_header_v8_final_anonymized_company.html)

GitHub displays HTML source instead of executing it. Open the file page, select **Download raw file**, and open the downloaded file in a browser. The demo is a self-contained experience prototype using local mock data; it does not call production services.

### Suggested walkthrough

1. Complete or edit passenger information.
2. Explore baggage, meal, and seat options.
3. Filter services by passenger and flight segment.
4. Add and remove options and observe conflict-handling behavior.
5. Review how selections update the cart and price summary.
6. Continue through the checkout states and validation messages.

## The product problem

Selling a fixed catalogue on a single airline storefront is relatively straightforward. A multi-provider travel platform faces a different problem: each airline or provider may expose services with different codes, structures, descriptions, prices, eligibility rules, and levels of detail.

The product challenge was therefore not simply to display a list of extras. It was to define an experience that could:

- absorb provider variation without exposing technical complexity to agency users;
- preserve passenger and flight-segment applicability;
- present baggage, meals, seats, and future service categories consistently;
- keep selection, pricing, payment, and booking state understandable;
- support multi-segment and round-trip journeys without losing context.

## Product perspective

The work connected three distinct models:

1. **Provider model** — external airline-retailing or provider-specific messages.
2. **Canonical platform model** — normalized service definitions, associations, pricing, and source references.
3. **Experience model** — user-facing categories, labels, filters, selection rules, and cart presentation.

```mermaid
flowchart LR
    EXT[Airline Provider API] --> ADP[Provider Adapter]
    ADP --> MAP[Validation and Normalization]
    MAP --> CORE[Canonical Ancillary Model]

    STORE[(Booking Context Store)] <--> CORE
    CORE --> EXP[Experience API]
    EXP --> UI[B2B Checkout UI]
    CORE <--> PAY[Payment Gateway]

    CORE --> ELG[Passenger and Segment Eligibility]
    CORE --> PRC[Price and Currency]
    CORE --> CAT[Category and Display Rules]

    ELG --> UI
    PRC --> UI
    CAT --> UI
```

[Open the Mermaid source](diagrams/system-context.mmd)

## From backend reality to UX behavior

| Technical or data reality | Product decision | Experience behavior |
|---|---|---|
| Providers can return different codes and content structures | Normalize into shared service categories | Consistent baggage, meal, and seat sections |
| A service may apply to only some travellers | Preserve passenger associations | Passenger-aware filters and service cards |
| Applicability can differ by flight segment | Preserve segment associations | Outbound, inbound, and segment-level context |
| Round-trip and single-direction options may overlap | Define replacement rules | Conflicting selections are updated with feedback |
| Price and availability may change after discovery | Revalidate before commitment | Quote/review step before final submission |
| Seat inventory has spatial and availability constraints | Use a dedicated interaction model | Seat map with occupied, available, selected, and premium states |
| Selected services alter the booking total | Keep price impact visible | Synchronized cart and price summary |
| Traveller data can be required for eligibility | Gate invalid selections | Clear validation and recovery guidance |

## End-to-end ancillary servicing

The following sequence is an anonymized reconstruction of the designed happy path. Component and operation names are functional portfolio labels, not production contracts or a claim of a schema-certified NDC implementation.

```mermaid
sequenceDiagram
    actor User as Agency User
    participant UI as B2B Checkout UI
    participant EXP as Experience API
    participant CORE as Ancillary Orchestrator
    participant STORE as Booking Context Store
    participant ADP as Provider Adapter
    participant AIR as Airline Retailing API
    participant PAY as Payment Gateway

    User->>UI: Open ancillary selection
    UI->>EXP: Request services for reservation
    EXP->>CORE: Retrieve applicable ancillary options

    CORE->>STORE: Load passengers, segments, and booking references
    STORE-->>CORE: Booking context

    CORE->>ADP: Retrieve current provider order
    ADP->>AIR: Retrieve current order
    AIR-->>ADP: Current order and service state
    ADP-->>CORE: Provider order response
    CORE->>STORE: Reconcile booking context

    CORE->>ADP: Request applicable services
    ADP->>AIR: ServiceListRQ with order and filter context
    AIR-->>ADP: ServiceListRS with service offers
    ADP-->>CORE: Provider-specific service response

    CORE->>CORE: Filter and normalize service definitions
    CORE->>CORE: Map passenger and segment applicability
    CORE->>STORE: Store normalized references

    CORE-->>EXP: Canonical ancillary catalogue
    EXP-->>UI: UI-ready service definitions
    UI-->>User: Display passenger- and segment-aware options

    User->>UI: Select services and continue
    UI->>EXP: Submit selection and payment choice
    EXP->>CORE: Process selected services

    CORE->>CORE: Validate offer references, quantity, and applicability
    CORE->>ADP: Request current quote
    ADP->>AIR: Quote selected services
    AIR-->>ADP: Confirmed price and service references
    ADP-->>CORE: Quote result

    CORE->>PAY: Authorize quoted amount
    PAY-->>CORE: Authorization result

    CORE->>ADP: Commit order update
    ADP->>AIR: Add selected services
    AIR-->>ADP: Updated order response
    ADP-->>CORE: Provider update result

    CORE->>ADP: Retrieve authoritative order state
    ADP->>AIR: Retrieve updated order
    AIR-->>ADP: Updated services and associations
    ADP-->>CORE: Authoritative provider state
    CORE->>STORE: Reconcile internal booking state

    CORE->>PAY: Capture authorized amount
    PAY-->>CORE: Capture result

    CORE-->>EXP: Updated reservation and service state
    EXP-->>UI: Confirmation and updated totals
    UI-->>User: Show completed ancillary purchase
```

[Open the Mermaid source](diagrams/ancillary-servicing-sequence.mmd)

## Delivery decomposition

The experience was decomposed according to behavioral and data dependencies rather than screen sections alone.

```mermaid
flowchart LR
    P[Passenger Information] --> D[Form State and Validation]
    D --> A[Ancillary Catalogue and Cart]
    A --> S[Price Summary]
    A --> SEAT[Seat Selection]
    S --> LOCK[Selection Locks and Warnings]
    SEAT --> LOCK
    LOCK --> CTA[Contact and Checkout Actions]
    CTA --> PAY[Payment and Submission]
```

[Open the Mermaid source](diagrams/story-dependencies.mmd)

## Key product decisions

- Treat passenger and segment identity as part of every applicable service selection.
- Keep provider codes and structures out of the user-facing information hierarchy.
- Handle overlapping one-way and round-trip choices as explicit replacement behavior.
- Give seats a dedicated interaction model while keeping their price impact inside the same cart.
- Preserve selections while users move between passengers, segments, and service filters.
- Revalidate price and provider references before committing the booking change.
- Separate payment authorization from capture around the provider order update.
- Retrieve the authoritative provider state after mutation before presenting final confirmation.

The reasoning and alternatives are documented in [Product decisions](product-decisions.md).

## Edge cases considered

- incomplete passenger information;
- service not applicable to a selected traveller or segment;
- overlapping single-direction and round-trip options;
- occupied or unavailable seats;
- price or availability changing between discovery and submission;
- provider timeout or empty service catalogue;
- payment authorization failure;
- provider order update failure after authorization;
- payment capture failure after the booking update;
- provider state differing from the local booking state.

The prototype demonstrates selected UX cases. Failure and recovery scenarios not visible in the prototype are documented as **design considerations**, not as implemented production behavior.

## NDC alignment

IATA NDC 21.3 was used as a reference version for examining the concepts behind service discovery, service definitions, passenger and segment associations, offer items, order servicing, and seat availability. Internal application contracts used their own names and were mapped at the provider boundary.

This case study is **NDC-aligned**, not presented as NDC-certified or schema-validated. See [NDC alignment and scope](ndc-alignment.md).

## Example delivery artefacts

- [Selected acceptance criteria and QA scenarios](acceptance-criteria.md)
- [Product decisions and trade-offs](product-decisions.md)
- [NDC concept alignment](ndc-alignment.md)
- [Editable Mermaid diagrams](diagrams/)

## Measurement plan

No production performance figures are disclosed or invented. A suitable measurement framework would include:

| Metric | Purpose |
|---|---|
| Ancillary attach rate | Measure adoption among eligible bookings |
| Mapping coverage | Track how much incoming provider content maps cleanly |
| Unmapped-service rate | Detect catalogue gaps and new provider codes |
| Selection failure rate | Identify UX or eligibility friction |
| Quote-change rate | Monitor price/availability changes before commitment |
| Passenger-segment mismatch rate | Detect invalid service associations |
| Provider response latency | Understand discovery and checkout performance |
| Checkout abandonment after interaction | Measure friction introduced by the ancillary step |

## Scope and evidence

| Classification | Meaning in this case study |
|---|---|
| Demonstrated | Visible and interactive in the HTML prototype |
| Modelled | Represented in the prototype's local data and state model |
| Designed | Defined as a product or integration behavior in the reconstructed specification |
| Considered | Recommended edge case or control, not claimed as implemented |
| NDC-aligned | Conceptually mapped to the examined IATA model without certification claims |
