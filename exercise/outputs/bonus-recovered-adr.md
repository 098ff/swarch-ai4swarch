Agent: AI assistant using Copilot SDK in VS Code; Model: not provided

# ADR-BONUS: Use synchronous gRPC with a shared protocol definition

- **Status:** Reconstructed
- **Date:** 2026-10-25
- **Deciders:** Not evidenced in the provided files.

## Context and problem

The provided `demo.proto` defines request/response RPCs for Cart, Recommendation, Product Catalog, Shipping, Currency, Payment, Email, Checkout, and Ad services. The frontend creates gRPC client connections to several services (`frontendServer`, `main`), while checkout calls Cart, Product Catalog, Shipping, Currency, Payment, and Email clients during `PlaceOrder` (`checkoutService.PlaceOrder` and its helper methods).

The checkout flow consumes each result as it proceeds: it prepares items and shipping costs before charging, ships after charging, and then returns an order response. Errors from several calls are returned to the caller. This supports describing the observed calls as synchronous request/response interactions. It does not establish that every interaction in the whole system is synchronous or uses this file.

The evidence shows the design, but not why it was chosen. The rationale below is therefore reconstructed rather than documented fact.

## Decision drivers

- **[Inferred]** Keep service contracts consistent and understandable across the services represented in the shared schema.
- **[Inferred]** Make request/response calls convenient where a caller needs a downstream result to continue a user-facing workflow.
- **[Inferred]** Prefer direct request-level error and result handling for operations such as placing an order.

## Considered options

1. Use synchronous gRPC RPCs defined together in `demo.proto`.
2. Use synchronous gRPC with a separate protocol definition owned by each service.
3. Use asynchronous messages/events for service-to-service workflows.

### Pros and cons of the options

| Option | Pros | Cons |
|---|---|---|
| Shared `demo.proto` with synchronous gRPC | **[Inferred]** A common schema can make request/response types and service contracts easier to discover together. Direct replies suit the observed checkout steps that consume downstream results. | **[Inferred]** A shared schema may couple service changes and require coordination. Synchronous dependencies can extend request latency and make a caller's progress depend on downstream availability. |
| Separate per-service gRPC definitions | **[Inferred]** Service teams could change contracts independently, improving ownership and modifiability. | **[Inferred]** Cross-service contracts may be harder to discover and keep consistent; callers still wait for results and remain dependent on downstream availability. |
| Asynchronous messaging/events | **[Inferred]** Decoupling producers from immediate consumer responses could improve temporal independence and resilience to brief consumer outages. | **[Inferred]** The observed checkout workflow needs values such as cart contents, prices, shipping quotes, and payment results to produce its response. An asynchronous design would add workflow/state-handling complexity and could delay completion feedback. |

## Decision

Reconstruct the decision as using synchronous gRPC request/response calls for the represented service APIs, with their service and message declarations centralized in `demo.proto`.

**[Inferred rationale]** The visible checkout sequence is a plausible reason to favor synchronous calls: it uses downstream results to calculate totals, charge a card, arrange shipping, and construct an order response. A shared schema plausibly makes those cross-service contracts easier to discover and use consistently. The evidence does not document these motivations, nor establish that they were the actual reasons for the original design.

## Consequences

- **Positive [Inferred]:** A shared schema can improve contract discoverability and consistency (quality attribute: modifiability).
- **Positive [Evidenced]:** The checkout handler can use returned values and errors to construct or fail an order response (quality attribute: functional correctness); see `checkoutService.PlaceOrder` and its helper methods.
- **Negative [Inferred]:** Synchronous downstream calls can add cumulative latency to a workflow that performs several calls in sequence (quality attribute: performance).
- **Negative [Evidenced]:** Checkout propagates failures from required downstream operations, so those operations affect order completion (quality attribute: availability); see `checkoutService.PlaceOrder`, `prepareOrderItemsAndShippingQuoteFromCart`, `chargeCard`, and `shipOrder`.
- **Risks and mitigations [Inferred]:** A shared contract can create coordination costs when services evolve (quality attribute: modifiability). Review schema changes for compatibility and coordinate updates among affected service owners.

## Validation

**[Inferred proposal]** Trace representative frontend requests and order placements to measure end-to-end latency, per-service latency, and downstream error rates. Review schema changes for compatibility and verify that changes to a service contract do not unintentionally break its callers. No such validation results or thresholds are evidenced in the provided files.

## Claim check

| # | Claim | Type | Evidence |
|---|---|---|---|
| 1 | `demo.proto` defines request/response RPCs for the listed services. | Evidenced | `system/online-boutique/protos/demo.proto` (`CartService` through `AdService`; `rpc ... returns (...)`) |
| 2 | The frontend establishes gRPC client connections to multiple service addresses. | Evidenced | `system/online-boutique/src/frontend/main.go` (`frontendServer`, `main`, `mustConnGRPC`) |
| 3 | Checkout calls downstream RPCs and uses returned values/errors while placing an order. | Evidenced | `system/online-boutique/src/checkoutservice/main.go` (`PlaceOrder`, `prepareOrderItemsAndShippingQuoteFromCart`, `quoteShipping`, `getUserCart`, `prepOrderItems`, `chargeCard`, `shipOrder`) |
| 4 | The design was selected to provide consistent, discoverable contracts and support result-dependent request workflows. | Inferred | Plausible interpretation of `demo.proto` and `checkoutService.PlaceOrder`; no rationale is stated in the provided files. |
| 5 | Separate service definitions could improve independent ownership but make cross-service contracts harder to discover. | Inferred | Alternative and trade-off; not demonstrated in the provided files. |
| 6 | Asynchronous messaging could improve temporal decoupling but add workflow/state-handling complexity for the observed checkout sequence. | Inferred | Alternative and trade-off based on the result dependencies in `checkoutService.PlaceOrder`; no message-based design is shown. |
| 7 | A shared schema may improve discoverability while increasing coordination costs; synchronous calls may add latency and expose completion to downstream availability. | Inferred | Potential consequences of the reconstructed decision; no measured impact is provided. |
| 8 | Tracing latency, errors, and schema compatibility would help validate the decision. | Inferred | Proposed validation only; no results or thresholds are evidenced in the provided files. |
