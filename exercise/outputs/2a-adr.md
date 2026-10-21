Agent: AI assistant using Copilot SDK in VS Code; Model: not disclosed.

# ADR-0002: Use event-driven post-order processing

- **Status:** Proposed
- **Date:** 2026-10-22
- **Deciders:** Not specified

## Context and problem

After an order is placed, Online Boutique must execute three post-order actions:

1. Send an order confirmation email via MailFast, a new external email provider replacing the mock `emailservice` logic.
2. Award loyalty points through a new `loyaltyservice` (to be built).
3. Record an "order placed" analytics event for the marketing team.

Marketing expects more post-order actions next year (SMS, fraud scoring, recommendations update). We must choose an architecture that accommodates future actions without tightly coupling the checkout flow to individual downstream operations. No data regarding protocol, throughput, latency targets, SLAs, or messaging mechanisms was provided.

## Decision drivers

- Support adding future post-order actions with minimal impact on the checkout flow (modifiability).
- Minimize dependency on downstream availability and latency for order placement responses (availability, performance).
- Prevent failure in one action from blocking other actions (availability, reliability).
- Control infrastructure complexity and operational overhead (cost, operability).

## Considered options

1. Synchronous direct calls to MailFast, `loyaltyservice`, and analytics from the order placement service.
2. Publish an `OrderPlaced` event via a durable event transport, processed by decoupled consumers.
3. Introduce a post-order orchestrator service that coordinates downstream calls.

### Pros and cons of the options

| Option | Pros | Cons |
|---|---|---|
| 1. Direct synchronous calls | Simple and straightforward; clear execution flow and immediate feedback. | Checkout latency and availability become tightly coupled to all downstream services; adding actions adds synchronous dependencies. |
| 2. Event publishing with decoupled consumers | Decouples order placement from post-order workflows; easily accommodates new consumers; downstream outages do not block checkout. | Requires managing durable event transport; introduces eventual consistency and potential duplicate delivery; requires idempotency. |
| 3. Post-order orchestrator | Centralizes coordination and retry policies; order service does not need to know all endpoints. | Orchestrator becomes a single point of complexity/bottleneck; adding actions still requires updating orchestrator configuration. |

## Decision

Choose **Option 2**: Upon successful order placement, publish an `OrderPlaced` event via a durable event transport, allowing independent consumers to handle MailFast email delivery, loyalty point allocation, and analytics dispatch. Replacing mock `emailservice` logic will occur inside the email consumer rather than via synchronous calls from checkout.

The specific event transport technology, event schema, and reliable publishing mechanisms are not finalized in this initial proposal.

## Consequences

- **Positive:** New consumers can be added without modifying order placement logic (modifiability).
- **Positive:** Downstream latency or outages in MailFast/loyalty/analytics do not impact order placement latency or availability (availability, performance).
- **Negative:** Requires provisioning, monitoring, and operating an event transport and backlog queues (cost, operability).
- **Negative:** Eventual consistency; post-order processing occurs asynchronously; consumers must handle duplicate events idempotently (consistency, reliability).
- **Risks and mitigations:** Risk of message loss if order succeeds but event publication fails; requires a resilient publishing mechanism (reliability).

## Validation

- Verify that order placement generates `OrderPlaced` events and all consumers execute their respective tasks.
- Verify that downstream consumer failures or delays do not block checkout response times.
- Test duplicate event delivery to ensure no duplicate emails or loyalty points are awarded.
- Measure checkout latency, consumer backlog, and end-to-end completion times once SLA targets are agreed upon.

## Open questions

- Which datastore/mechanism will guarantee atomic event publication with order persistence?
- Which specific event broker should be chosen (Kafka, RabbitMQ, etc.)?
- What is the payload schema for `OrderPlaced`?
- What are MailFast's exact API contracts, authentication requirements, and rate limits?
- How should idempotency be enforced in `loyaltyservice`?
