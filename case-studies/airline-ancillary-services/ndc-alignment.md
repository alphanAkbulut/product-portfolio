# IATA NDC 21.3 Alignment and Scope

## Why NDC is relevant

The case study addresses a domain represented in IATA airline-retailing standards: discovering and selling services associated with an offer or an existing order while preserving traveller, flight, price, and fulfillment context.

IATA NDC 21.3 was examined as a reference version. It is not presented as the only version or as proof that the prototype executes NDC XML.

## Concept mapping

| Product concern | Closest NDC 21.3 concept | Portfolio representation |
|---|---|---|
| Discover applicable ancillary services | `ServiceListRQ/RS` | Provider service discovery in the sequence diagram |
| Describe a non-flight service | `ServiceDefinition` | Canonical service category, code, name, and description |
| Represent a sellable service | A-la-carte offer/item | Selectable baggage, meal, or seat option |
| Associate a service with a traveller | Passenger reference | Passenger-aware cards, filters, and cart rows |
| Associate a service with travel | Passenger-segment or flight association | Segment and direction labels |
| Retrieve current order context | `OrderRetrieveRQ` with an order-view response | Retrieve and reconcile current provider state |
| Quote additions to an existing order | `OrderQuoteRQ` with the applicable reshop response | Revalidate selected services and price |
| Commit an order modification | `OrderChangeRQ` with an updated order view | Add selected services and retrieve authoritative state |
| Shop for seats | `SeatAvailabilityRQ/RS` | Segment-specific seat map and priced seat selection |
| Support service delivery | Service and entitlement state | Outside the interactive prototype's primary scope |

## Internal contracts versus standard messages

The original professional workflow used application-specific request and response contracts between internal services. Their names should not be interpreted as IATA-standard message names.

At the provider boundary, internal operations were conceptually associated with airline-retailing functions such as retrieving an order, discovering services, quoting selections, and committing an order change.

This distinction is intentional:

```text
Internal application contract
        ↓
Provider adapter and mapping
        ↓
Provider-specific or NDC-aligned contract
```

It prevents external schema details from leaking into every internal consumer and allows the experience layer to depend on a stable canonical model.

## Version and claim boundaries

- NDC 21.3 is the examined reference version, not a universal contract for every provider.
- Element names, cardinalities, and workflows can vary by NDC version and provider implementation.
- The HTML prototype uses local mock data and does not send or validate XML messages.
- No formal IATA certification or schema compliance is claimed.
- `NDC-aligned` means the product and domain relationships map conceptually to the examined standard.
- Public diagrams use functional names and do not expose exact production contracts or topology.

## References

- [IATA Airline Retailing XSD Viewer](https://retailing.iata.org/tools/xsd_viewer/)
- [IATA NDC 21.3 Implementation Guide](https://guides.developer.iata.org/docs/21-3_ImplementationGuide.pdf)

