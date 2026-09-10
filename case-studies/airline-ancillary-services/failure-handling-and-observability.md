# Failure Handling and Observability

## Purpose and evidence boundary

Ancillary checkout depends on internal data, airline-provider APIs, and payment services. A useful product design therefore needs to define not only the success path, but also what the customer sees when one of those dependencies returns incomplete data, fails, slows down, or produces an uncertain result.

This document describes a **recommended product and integration design** for the reconstructed portfolio case study. It does not claim that a particular logging stack, alert, retry policy, error catalogue, or recovery mechanism was implemented in production.

## Product principles

1. Never expose raw provider, database, or payment error messages to the customer.
2. Normalize dependency-specific failures into a small canonical error model.
3. Preserve the traveller's selections when continuing is safe.
4. Prefer partial availability over blocking the complete checkout when the missing data is non-critical.
5. Suppress an affected service when price, currency, traveller, segment, or eligibility data cannot be trusted.
6. Offer retry only when the operation is safe to repeat.
7. Treat an unknown payment or booking outcome as a reconciliation problem, not as an invitation to submit the same transaction again.
8. Give support and operations a non-sensitive correlation reference without exposing internal details in the interface.

## Canonical error taxonomy

| Source and stage | Example condition | Canonical category | Recommended system response | Recommended customer experience |
|---|---|---|---|---|
| Outbound request validation | Required provider parameter is missing, invalid, or internally inconsistent | `REQUEST_VALIDATION` | Reject before the provider call where possible; record the failed validation and owning stage | Ask for correction only when the customer can fix the input; otherwise show a general service-unavailable message |
| Airline-provider availability | Timeout, rate limit, load-related degradation, connection failure, or temporary server error | `EXTERNAL_TRANSIENT` | Apply a bounded retry only to safe operations; stop when the retry budget is exhausted | Preserve selections and offer retry or continuation without extras when safe |
| Airline-provider business rule | The service is no longer eligible or available for the selected traveller or segment | `EXTERNAL_BUSINESS_RULE` | Map the known response to a stable internal reason | Explain the actionable outcome without displaying the provider code |
| Airline-provider contract | Unknown error code, malformed payload, missing required field, or incompatible response shape | `EXTERNAL_UNKNOWN` | Use a safe fallback category; retain the correlation reference and flag the contract gap | Do not display raw response content; show a neutral message and a safe next step |
| Internal data and mapping | A provider value has no database match or cannot be converted to the canonical service model | `DATA_MAPPING` | Count the mapping gap; continue with valid items when possible; suppress only affected unsafe items | Keep the rest of the catalogue usable and avoid presenting incomplete service information |
| Offer and quote lifecycle | Price, availability, or offer reference changes before commitment | `OFFER_STALE` | Refresh or requote and require review before committing | Show the updated price or availability and ask the customer to confirm again |
| Payment business outcome | Known decline or customer-correctable payment response | `PAYMENT_DECLINED` | Map to an actionable payment category without storing sensitive payment data | Ask the customer to check the payment details or choose another method |
| Payment technical outcome | Gateway timeout, temporary unavailability, or transport failure with a known unsuccessful result | `PAYMENT_TECHNICAL` | Preserve checkout state and retry only under the payment policy | Keep the selection and provide a safe retry or alternative-payment action |
| Payment uncertain outcome | The request was sent but the final authorization or capture state is unknown | `PAYMENT_UNKNOWN` | Reconcile payment status before another attempt; prevent duplicate submission | Explain that the status is being checked and do not invite an immediate repeat payment |
| Booking-state reconciliation | Provider order state and internal booking state disagree after a mutation | `STATE_CONFLICT` | Retrieve authoritative state, compare it with the internal record, and route unresolved cases for operational review | Avoid presenting a false confirmation; show a neutral pending/review state when necessary |

Provider-specific and payment-specific error-code tables would be separate implementation artefacts. Their role is to map many external codes and messages into these stable categories, not to drive UI copy directly.

## Critical versus non-critical mapping gaps

Not every mapping problem should make the entire ancillary step unavailable.

| Missing or unmapped information | Recommended treatment |
|---|---|
| Marketing description or optional display attribute | Use an approved fallback label when it remains unambiguous |
| One unknown service among otherwise valid services | Exclude or safely generalize the affected item; continue showing valid services |
| Price or currency | Do not allow selection of the affected item |
| Traveller or flight-segment association | Do not display the service as selectable until applicability is known |
| Eligibility or fulfilment rule | Suppress the affected service and record the mapping gap |
| Entire catalogue cannot be normalized | Show a service-unavailable state while allowing the core booking flow to continue when permitted |

This distinction protects conversion without allowing incomplete data to create an invalid order.

## UI behavior contract

The interface should be driven by the canonical category and recovery policy rather than by the originating system.

| UI decision | Questions the product contract must answer |
|---|---|
| Message | Is the problem understandable and actionable without technical detail? |
| Retry | Is repeating the operation safe, and has the retry budget been exhausted? |
| Selection state | Should existing passenger, segment, seat, meal, or baggage selections be retained? |
| Checkout continuity | Can the customer continue without the affected service? |
| Support reference | Does the case need a non-sensitive reference for later investigation? |
| Pending state | Is the outcome uncertain enough that confirmation must wait for reconciliation? |

Example customer-facing messages are intentionally generic and would require content-design and localization review:

- **Temporary dependency problem:** “Extra services are temporarily unavailable. Try again or continue without extras.”
- **Affected item unavailable:** “This option is not available right now. Your other selections are unchanged.”
- **Price or availability changed:** “The price or availability changed. Review the updated selection before continuing.”
- **Known payment problem:** “Payment could not be completed. Check the details or choose another payment method.”
- **Unknown payment state:** “We are checking the payment status. Please do not submit another payment yet.”

## What should be logged

Each stage should emit a structured event that supports investigation without exposing customer or payment data. Useful fields include:

- timestamp and environment;
- correlation ID and operation name;
- processing stage and dependency type;
- anonymized provider alias;
- outcome and canonical error category;
- retryability and retry count;
- duration and dependency latency;
- mapped, unmapped, accepted, and rejected item counts where relevant;
- anonymized or hashed booking reference;
- UI message key and selection-retention decision;
- reconciliation status for uncertain payment or booking outcomes.

Illustrative, non-production example:

```json
{
  "timestamp": "2026-01-15T09:42:18Z",
  "correlation_id": "demo-corr-7f3a",
  "operation": "ancillary.discovery",
  "stage": "provider_response_mapping",
  "dependency_type": "airline_provider",
  "provider_alias": "provider-a",
  "outcome": "degraded_success",
  "error_category": "DATA_MAPPING",
  "retryable": false,
  "latency_ms": 842,
  "mapped_item_count": 11,
  "unmapped_item_count": 1,
  "booking_reference_hash": "sha256:illustrative-value",
  "ui_message_key": "SERVICE_DETAILS_UNAVAILABLE"
}
```

## What should not be logged

- traveller names, contact details, or identity-document data;
- card number, security code, unmasked payment token, or raw gateway payload;
- authentication credentials, API keys, or session tokens;
- complete booking, offer, order, or payment identifiers when a masked or hashed value is sufficient;
- unrestricted raw provider requests and responses;
- internal stack traces in customer-facing responses.

Sensitive payload capture, if ever required for controlled troubleshooting, would need separate access, retention, masking, and deletion policies.

## Monitoring and product signals

| Signal | What it helps detect |
|---|---|
| Provider timeout and transient-error rate | External degradation, load, or connectivity problems |
| Response latency by operation and provider alias | Slow discovery, requote, or order servicing |
| Unknown external-error rate | New provider codes or contract changes |
| Mapping coverage and unmapped-service rate | Missing catalogue configuration and normalization gaps |
| Degraded-success rate | How often customers receive a partial catalogue |
| Price or availability change rate | Offer volatility between discovery and commitment |
| Payment-unknown rate | Cases that must be reconciled before retry |
| Booking-state mismatch rate | Provider/internal consistency problems |
| Retry recovery rate | Whether bounded retries improve outcomes |
| Checkout abandonment following an error | Customer impact rather than technical volume alone |

Thresholds are deliberately omitted because they require real baselines, service-level objectives, and operational ownership.

## Investigation workflow

1. Start from the customer-visible time and correlation reference.
2. Identify the failed stage before selecting a root-cause theory.
3. Separate request-validation evidence from provider, mapping, payment, and state-reconciliation evidence.
4. Check whether the outcome was a complete failure, degraded success, or an uncertain transaction.
5. Compare the canonical category with the originating dependency response without exposing the raw response outside controlled access.
6. For payment or order mutations, confirm the authoritative state before recommending a retry.
7. Record observed evidence separately from interpretation and list competing causes in probability order.
8. Feed new, repeatable provider or payment errors into the relevant mapping table after review.

## Failure-flow diagram

The accompanying [failure and recovery flow](diagrams/failure-recovery-flow.mmd) visualizes how validation, mapping, transient dependency failures, unknown responses, payment uncertainty, and UI recovery decisions fit together.
