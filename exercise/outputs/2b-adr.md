Agent: AI assistant using Copilot SDK in VS Code; Model: not disclosed.

# ADR-0002: Use RabbitMQ for post-order processing

- **Status:** Proposed
- **Date:** 2026-10-22
- **Deciders:** Not specified

## Context and problem

Requirement R-12 states that after an order is placed, the system must send an order confirmation email via MailFast (replacing mock `emailservice` logic), award loyalty points via a new `loyaltyservice`, and record an "order placed" analytics event for marketing. Marketing also anticipates adding SMS, fraud scoring, and recommendations updates next year (R-12 in `docs/scenario.md`).

Currently, `checkoutservice.PlaceOrder` sequentially calls `paymentservice`, `shippingservice`, and `emailservice` synchronously over gRPC (`src/checkoutservice/main.go`). If email delivery fails, it logs a warning but proceeds with order success. The SLA requires p95 latency of `PlaceOrder` to remain ≤ 500 ms (currently ~300 ms), whereas MailFast has a p95 of 1.5s, an error rate of ~0.5%, and a strict rate limit of 50 emails/s. Peak traffic during the 11.11 sale reaches 200 orders/s. Furthermore, loyalty points must be awarded within 1 minute and strictly once (no double points), and checkout must never fail due to downstream email, loyalty, or analytics outages (`docs/scenario.md`).

The codebase generates an `orderID` in `PlaceOrder` but lacks persistent order storage or a durable outbox (`src/checkoutservice/main.go`).

## Decision drivers

1. **Availability:** Failures in MailFast, `loyaltyservice`, or analytics must never cause checkout to fail (R-12 in `docs/scenario.md`).
2. **Performance:** Maintain `PlaceOrder` p95 latency ≤ 500 ms; calling MailFast synchronously (p95 1.5s) violates this SLA.
3. **Correctness:** Award loyalty points within 1 minute without duplicate awards.
4. **Scalability:** Handle 200 orders/s peak bursts while respecting MailFast's 50 emails/s rate limit.
5. **Modifiability & Operations:** Support adding future post-order actions; leverage existing team familiarity with RabbitMQ (2 of 4 developers) and preference for open-source, self-hosted components on Kubernetes.

## Considered options

1. Direct synchronous calls to MailFast, `loyaltyservice`, and analytics from `checkoutservice`.
2. Publish post-order events via RabbitMQ to independent consumer services.
3. Introduce a synchronous post-order orchestrator service.

### Pros and cons of the options

| Option | Pros | Cons |
|---|---|---|
| Synchronous calls | Similar to existing gRPC patterns in `checkoutservice`; straightforward flow. | MailFast latency (1.5s) violates the 500ms SLA; tight coupling; cannot absorb 200 orders/s peak with a 50 emails/s limit. |
| RabbitMQ with decoupled consumers | Decouples checkout latency from downstream processing; queues buffer 200 orders/s bursts; extensible; matches team skills and self-hosted preference. | Adds broker operational overhead on Kubernetes; requires durable publishing, retry, rate limiting, and idempotency mechanisms; outbox pattern not yet in codebase. |
| Synchronous orchestrator | Hides downstream endpoints from `checkoutservice`. | Still couples order success/latency to external endpoints; cannot buffer load bursts; adds another synchronous service. |

## Decision

Choose **RabbitMQ with decoupled consumers per post-order action**. This ensures `checkoutservice` responds without waiting for MailFast, loyalty, or analytics, and accommodates future actions (SMS, fraud scoring) while respecting rate limits and team skillsets.

Events will include unique `orderID`s acting as idempotency keys for `loyaltyservice`. Durable queues and retries will protect in-flight messages during consumer restarts.

Before deployment, a durable transactional outbox or equivalent mechanism must be established to ensure reliable publishing without risking order consistency.

## Consequences

- **Positive:** `PlaceOrder` latency is unaffected by MailFast delays (**performance**).
- **Positive:** Queues buffer bursts when ingress (200 orders/s) exceeds MailFast rate limits (50 emails/s) (**scalability, availability**).
- **Positive:** Decoupled consumers enable adding future workflows without modifying checkout (**modifiability**).
- **Negative:** Adds RabbitMQ and persistent storage components to deploy, monitor, and maintain on Kubernetes (**operability, cost**).
- **Negative:** Asynchronous processing means eventual consistency; email and analytics are deferred (**timeliness**).
- **Risks and mitigations:** Duplicate deliveries could cause duplicate loyalty points; consumers must deduplicate using `orderID` (**correctness**).
- **Risks and mitigations:** Message loss if checkout confirms before events are durably persisted; requires an outbox pattern (**reliability**).
- **Risks and mitigations:** Exceeding MailFast's 50 emails/s limit; consumer must enforce rate limiting (**compliance**).

## Validation

- Load test `PlaceOrder` at 200 orders/s to verify p95 latency ≤ 500 ms even during downstream outages.
- Simulate MailFast latency (p95 1.5s, p99 8s, 0.5% failure, 50/s limit); verify checkout SLA and consumer throttling.
- Test retry and redelivery to verify loyalty points are awarded within 1 minute and strictly once.
- Test broker restart and backlog recovery to ensure zero message loss.

## Open questions

- What datastore will be used for persistent orders and outbox events?
- How to handle publishing failures without violating checkout availability or losing events?
- What schema and dead-letter queue (DLQ) policies should be adopted?
- How will `loyaltyservice` persist idempotency records?
