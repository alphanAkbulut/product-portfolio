# Response and Error Mapping

## 1. Response-to-domain normalization

This mapping turns provider-specific response structures into a provider-neutral product model used by the experience layer.

```text
Provider response
→ adapter validation
→ canonical product model
→ eligibility and presentation rules
→ UI state
```

The canonical model must support the business concepts required by all intended providers. It should not inherit one provider's codes, cardinality, or state semantics as its universal contract.

### Example

Provider A and Provider B both return an ancillary option. Provider B adds an attribute the canonical model cannot classify. The response is valid, but the option cannot be safely priced or presented. The platform therefore keeps unrelated offers visible, suppresses the unsafe option, and records a `DOMAIN_MAPPING_GAP`.

## 2. Provider-error-to-product-error mapping

This mapping turns a provider's error signal into a stable internal category that determines the customer message, retry rule, transaction state, and operational action.

```text
Provider error code and context
→ error catalogue lookup
→ internal product category
→ customer message and UI state
→ retry, reconciliation, flag, or escalation decision
```

### Example

An error code that means “service unavailable” must not be mapped as “customer input invalid.” The first may allow a safe retry or continuation without the service. The second asks the customer to take an action. An incorrect match creates a misleading and potentially dead-end journey.

## Catalogue governance

Every catalogue entry should define:

- provider alias and source code pattern;
- internal category;
- whether the result is final, retryable, or uncertain;
- whether a retry is idempotent and within the permitted retry budget;
- approved customer message key;
- selection-retention rule;
- operational owner;
- flag and escalation conditions.

Unknown codes never silently inherit a convenient known category. Invalid matches are treated the same way: they enter a safe fallback path and create evidence for review.
