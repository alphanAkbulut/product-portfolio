# Error Taxonomy

Each category is designed around the product impact, not merely the component that emitted the error.

| Category | Example evidence | Customer state | Safe default | Owner |
|---|---|---|---|---|
| `INPUT_VALIDATION` | Required or incompatible customer selection | Correctable | Highlight the affected input and explain what to change | Product / UI |
| `PROVIDER_TRANSIENT` | Timeout, rate limit, temporary unavailable response | Safe retry, safe continuation, or reconciliation | Preserve selections; retry only when the action is safe and deduplicated | Integration |
| `PROVIDER_BUSINESS_RULE` | Service no longer eligible for a passenger or segment | Not retryable without a changed selection | Explain the outcome without provider code | Product / integration |
| `DOMAIN_MAPPING_GAP` | Valid provider field has no safe canonical-model representation | Degraded | Suppress only the affected item; keep safe alternatives available | Platform / integration |
| `ERROR_CATALOGUE_GAP` | Provider error code has no internal catalogue entry or has an invalid match | Unknown; requires review | Use a neutral fallback and create review evidence | Integration / operations |
| `DATA_STATE_MISMATCH` | Internal data disagrees with authoritative downstream state | Uncertain | Reconcile before confirmation or reversal | Data / operations |
| `PAYMENT_STATE_UNCERTAIN` | Authorization or capture outcome cannot yet be confirmed | Pending or unknown | Do not claim success or failure; reconcile | Payment / operations |
| `UI_RENDERING_FAILURE` | The interface cannot render a safe product state | Safe fallback or operational follow-up | Protect the transaction; show a clear fallback and retain a support reference | UI / platform |

## Classification rule

The platform records both the technical source and the product category. For example, a Provider B response may be technically successful but create a `DOMAIN_MAPPING_GAP`; it must not be counted as a provider outage.

## Customer-message rule

The interface never displays raw provider responses, database messages, stack traces, endpoint names, or internal error identifiers. It displays only the amount of certainty that the platform has earned.

| Product category | Suitable customer guidance |
|---|---|
| Correctable input | “Please review the highlighted information.” |
| Temporary provider issue | “This option is temporarily unavailable. Your other selections are saved.” |
| Eligibility issue | “This option is not available for the selected traveller or journey.” |
| Unknown or uncertain outcome | “We are checking the status of your request. Please do not submit it again.” |
| No safe self-service route | “We need to review this request before it can continue.” |
