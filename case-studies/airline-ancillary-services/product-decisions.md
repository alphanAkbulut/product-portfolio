# Product Decisions and Trade-offs

This document captures the reasoning behind the Airline Ancillary Services experience. It focuses on how integration and domain constraints were translated into product behavior.

## 1. Normalize before presentation

**Problem:** Airline and provider responses may use different codes, structures, descriptions, and levels of detail for comparable services.

**Decision:** Introduce a canonical ancillary model between the provider adapter and the experience layer.

**Why:** UI components should not contain provider-specific branching for every source. A shared model also makes category rules, analytics, validation, and future-provider onboarding more manageable.

**Experience consequence:** Users see consistent categories and information hierarchy regardless of the content source.

## 2. Make passenger and segment context explicit

**Problem:** A service may be valid only for a specific traveller, flight segment, or direction.

**Decision:** Treat passenger and segment associations as part of the selection identity rather than optional display metadata.

**Why:** The same service code can represent different purchasable items when applied to different travellers or segments.

**Experience consequence:** Filters, route labels, traveller labels, and cart rows maintain selection context.

## 3. Replace conflicting directional selections

**Problem:** A traveller can encounter both single-direction and combined round-trip variants of the same service.

**Options considered:**

- allow all selections and risk duplicates;
- block the new selection without explanation;
- replace the overlapping selection and explain the change.

**Decision:** Use explicit replacement behavior with user feedback.

**Experience consequence:** Selecting a combined option removes overlapping one-way variants; choosing a one-way variant replaces the conflicting combined choice.

## 4. Use a dedicated seat interaction

**Problem:** Seat selection includes spatial position, occupancy, characteristics, passenger assignment, and segment-specific inventory.

**Decision:** Present seats through a seat map while keeping them inside the same ancillary cart and pricing model.

**Experience consequence:** Users receive a specialized interaction without losing a unified checkout summary.

## 5. Preserve state across filters

**Problem:** Moving between travellers, segments, or service categories can hide selected items and create uncertainty.

**Decision:** Filters change the view, not the underlying cart state.

**Experience consequence:** Selections remain visible in the summary and return when the relevant filter is selected again.

## 6. Gate selection when required traveller data is incomplete

**Problem:** Provider eligibility or booking updates can require completed traveller information.

**Decision:** Prevent invalid selection attempts and explain which prerequisite is missing.

**Experience consequence:** The interface provides actionable guidance instead of allowing a late, opaque provider failure.

## 7. Revalidate before commitment

**Problem:** Ancillary prices, availability, and offer references can change after initial discovery.

**Decision:** Validate the selected references and obtain a current quote before committing the order change.

**Experience consequence:** The final amount is based on a refreshed provider decision, with changes requiring review rather than being silently applied.

## 8. Separate payment authorization and capture

**Problem:** Capturing payment before the provider confirms the service can create refund and reconciliation work.

**Decision:** Authorize the amount, commit the provider order update, verify the resulting order state, and then capture.

**Experience consequence:** The customer receives confirmation only after the booking and payment states have been reconciled.

**Important:** Compensating behavior for authorization, provider-update, or capture failures is documented as a design consideration unless supported by implementation evidence.

## 9. Retrieve authoritative state after mutation

**Problem:** A mutation response may not contain the complete or final order representation required by the platform.

**Decision:** Retrieve the current provider order after the update and reconcile it with internal booking state.

**Experience consequence:** Final confirmation and downstream state are based on the authoritative provider view.

## 10. Preserve a deliberate path to continue without extras

**Problem:** Preventing checkout because no ancillary is selected creates unnecessary conversion friction, while silently discarding existing selections is unsafe.

**Decision:** Allow continuation without extras, but require confirmation when that action removes existing selections.

**Experience consequence:** The user remains in control and understands the consequence before continuing.
