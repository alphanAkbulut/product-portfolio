# Acceptance Criteria

## Provider-specific safe degradation

```gherkin
Scenario: One provider cannot return a usable ancillary option
  Given an eligible checkout contains options from more than one provider
  And Provider B returns a valid response that cannot be normalized safely
  When the platform builds the customer view
  Then only the affected Provider B option is withheld
  And unrelated safe options remain available
  And a DOMAIN_MAPPING_GAP event is recorded with a correlation reference
```

## Correct error-catalogue behavior

```gherkin
Scenario: A provider error code is unknown to the error catalogue
  Given a provider returns an unrecognised error code
  When the platform classifies the outcome
  Then it must not label the error as customer-correctable without evidence
  And it must use the approved unknown-error customer state
  And it must create an error-catalogue review flag
```

```gherkin
Scenario: A provider error code is mapped to an incorrect product category
  Given a provider error is matched to a catalogue entry
  And the mapped category conflicts with the provider error context
  When the platform validates the mapping decision
  Then it must use the safe fallback state instead of the incorrect category
  And it must not ask the customer to correct input without supporting evidence
  And it must create an error-catalogue review flag
```

## Uncertain transaction safety

```gherkin
Scenario: The payment or provider outcome is uncertain
  Given a customer action was sent to a downstream dependency
  And a final outcome cannot be confirmed within the safe response window
  When the customer view is updated
  Then the transaction is marked pending or unknown, not failed
  And the customer is asked not to submit the action again
  And reconciliation is scheduled before a final confirmation or reversal
```

## UI message integrity

```gherkin
Scenario: A technical error reaches the experience layer
  Given the platform has classified a technical error
  When a customer-facing message is selected
  Then no raw provider code, database message, endpoint, or stack detail is displayed
  And the message reflects the known certainty and available next action
```

## Escalation

```gherkin
Scenario: A critical risk threshold is reached
  Given the monitoring rules identify uncertain transaction outcomes with material impact
  When the critical condition is met
  Then an incident flag is created with affected scope and containment status
  And the designated operational owners are notified
  And a stakeholder update is prepared using only approved, non-sensitive evidence
```
