# Selected Acceptance Criteria and QA Scenarios

The following examples are anonymized and condensed from a larger product specification. They demonstrate the level of behavioral detail used to make the experience testable.

## Passenger prerequisite

```gherkin
Scenario: Prevent selection for an incomplete passenger
  Given a passenger is missing required identity information
  When the user attempts to select an ancillary for that passenger
  Then the selection must not be added
  And the interface must explain which prerequisite is incomplete
```

## Passenger and segment association

```gherkin
Scenario: Add a service for a passenger and flight segment
  Given the passenger information is complete
  And the service is applicable to the selected flight segment
  When the user adds the service
  Then the cart must identify the service, passenger, and segment
  And the ancillary subtotal and booking total must be updated
```

## Filter-state preservation

```gherkin
Scenario: Preserve selections when changing filters
  Given services have been selected for multiple passengers and segments
  When the user changes the passenger or segment filter
  Then existing selections must remain in the cart
  And returning to the original filter must show the previous selections
```

## Directional replacement

```gherkin
Scenario: Replace overlapping one-way selections with a round-trip option
  Given the same base service is selected separately for outbound and inbound travel
  When the user selects the equivalent round-trip option
  Then the overlapping one-way selections must be removed
  And the round-trip option must be added once
  And the user must be informed that the selections were updated
```

## Seat assignment

```gherkin
Scenario: Assign one seat per passenger and segment
  Given an available seat map for the selected flight segment
  When the user assigns an available seat to a passenger
  Then the seat must be associated with that passenger and segment
  And the price summary must include the seat price
  And selecting a replacement seat must remove the previous assignment
```

## Continue without ancillary services

```gherkin
Scenario: Confirm removal before continuing without selected services
  Given at least one ancillary is currently selected
  When the user chooses to continue without ancillary services
  Then the interface must explain that existing selections will be removed
  And the selections must remain unchanged until the user confirms
```

## Quote change

```gherkin
Scenario: Require review when the current quote differs
  Given the user has selected one or more services
  And the provider returns a changed price or availability during validation
  When the refreshed quote is received
  Then the previous amount must not be committed silently
  And the user must be shown the updated condition before continuing
```

This scenario represents a product requirement and should not be interpreted as proof of a particular production implementation.

## Suggested QA coverage

- Adult, child, and infant traveller combinations
- One-way, round-trip, and connecting itineraries
- Service applicable to one segment versus multiple segments
- Empty service response
- Unknown service code with safe fallback presentation
- Provider response timeout
- Seat becomes unavailable during selection
- Price changes between discovery and quote
- Payment authorization failure
- Provider update fails after payment authorization
- Capture failure after provider update
- Duplicate submission and idempotency behavior
- Provider and internal booking state mismatch

