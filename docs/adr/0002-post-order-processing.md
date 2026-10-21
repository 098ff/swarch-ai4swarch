# ADR-0002: Use RabbitMQ and Transactional Outbox for Post-Order Processing

- **Status:** Proposed
- **Date:** 2026-10-22
- **Deciders:** Software Architecture Team (Student / Architect)

## Context and problem

Requirement **R-12** dictates that upon successful order completion (`PlaceOrder`), Online Boutique must execute three post-order operations:
1. Send an order confirmation email via **MailFast** (a new third-party provider replacing the mock `emailservice`).
2. Award **loyalty points** via a newly developed `loyaltyservice`.
3. Dispatch an **"order placed"** analytics event for the marketing team.
Additionally, marketing plans to introduce further post-order actions next year (SMS notifications, fraud scoring, recommendations updates).

### Current System Facts and Constraints (`docs/scenario.md`):
- Currently, `checkoutservice.PlaceOrder` operates **synchronously over gRPC**, sequentially calling `paymentservice`, `shippingservice`, and `emailservice`. Its current p95 latency is ~300 ms, against an SLA requirement of **p95 ≤ 500 ms**.
- The upcoming 11.11 shopping festival is projected to reach a peak volume of **200 orders/second**.
- External provider **MailFast** exhibits high latency (p95 = 1.5 s, p99 = 8 s), an error rate of ~0.5%, and a strict rate limit of **50 emails/second**.
- Loyalty points must be awarded **within 1 minute** of checkout, strictly **once (Exactly-once outcome)**.
- Availability mandate: Order placement **must never fail** due to outages in MailFast, loyalty, or analytics services.
- The engineering team consists of 4 developers proficient in Go and gRPC, with 2 having prior **RabbitMQ** experience. The infrastructure runs on Kubernetes, with an organizational mandate preferring **open-source, self-hosted** solutions.

## Decision drivers

1. **Availability:** Downstream outages or latency spikes in MailFast, loyalty, or analytics must not impact order completion.
2. **Performance (Latency):** Preserve `PlaceOrder` p95 latency ≤ 500 ms (synchronous MailFast calls taking 1.5–8s would breach the SLA).
3. **Scalability & Rate Limiting:** Ingest peak bursts of 200 orders/s while strictly throttling downstream dispatch to MailFast at ≤ 50 req/s.
4. **Correctness (Idempotency):** Guarantee loyalty points are delivered within 1 minute with zero duplicate allocations under at-least-once delivery.
5. **Data Durability:** Ensure zero message loss if broker or services restart following successful payment.
6. **Modifiability & Operational Feasibility:** Enable seamless onboarding of future post-order consumers using technologies self-hostable on Kubernetes.

## Considered options

1. **Synchronous gRPC calls from `checkoutservice`**: Sequentially invoke MailFast, loyaltyservice, and analytics.
2. **Asynchronous Event-Driven Architecture via RabbitMQ with Transactional Outbox Pattern**: Write order and events atomically to an outbox, publish to RabbitMQ, and process via decoupled consumers.
3. **Synchronous Post-Order Orchestrator Service**: Centralize fan-out logic in a dedicated intermediate service.

### Pros and cons of the options

| Option | Pros | Cons |
|---|---|---|
| **1. Synchronous gRPC** | Simple implementation without new infrastructure. | Breaches latency SLA (1.5s > 500ms); cascades third-party outages to checkout; cannot absorb 200 orders/s bursts against a 50 req/s limit. |
| **2. RabbitMQ + Outbox (Selected)** | Decouples checkout latency (p95 < 500ms); queues buffer 200 orders/s bursts into 50 req/s streams; outbox guarantees durability; aligns with team RabbitMQ skills. | Adds operational overhead for deploying, monitoring, and maintaining RabbitMQ and outbox relay workers on Kubernetes. |
| **3. Synchronous Orchestrator** | Encapsulates downstream complexity away from `checkoutservice`. | Still couples order latency and failure to downstream availability; cannot natively buffer bursts exceeding rate limits. |

## Decision

Adopt **Option 2: Asynchronous Event-Driven Architecture using RabbitMQ combined with the Transactional Outbox Pattern**, structured as follows:

1. **Transactional Outbox in `checkoutservice`:**
   - Upon successful payment and shipping reservation, `checkoutservice` persists the order state and inserts an `OrderPlaced` event into a local transactional outbox table atomically before returning success to the user.
   - An asynchronous relay worker tails the outbox, publishes events to RabbitMQ with publisher confirms, and marks records processed, guaranteeing **At-least-once delivery** and zero message loss.
2. **RabbitMQ Topology & Isolated Queues:**
   - Utilize a Fanout Exchange (`order.v1.events`) bound to three dedicated, independent queues:
     - `order.email.queue` (consumed by Email Consumer for MailFast)
     - `order.loyalty.queue` (consumed by `loyaltyservice`)
     - `order.analytics.queue` (consumed by Analytics Consumer)
   - Guarantees fault isolation so failure in one consumer does not impede others.
3. **Dead-Letter Exchange (DLX) & Retry Policy:**
   - Each queue routes rejected messages to a Dead-Letter Queue (DLQ) after 3 exponential backoff retries, preventing poisoned messages from stalling consumer pipelines.
4. **Rate Limiting for MailFast:**
   - The Email Consumer implements a Token Bucket Rate Limiter capped at **45 requests/second** (providing a safety margin below MailFast's 50 req/s ceiling), using `order.email.queue` as a durable buffer during 200 orders/s peak surges.
5. **Idempotency for Loyalty Points:**
   - Every event includes the immutable `order_id`. `loyaltyservice` applies a unique constraint index on `order_id` in its datastore. Duplicate deliveries trigger a no-op success response, achieving an **Exactly-once outcome**.

## Consequences

- **Positive:**
  - **Performance:** `PlaceOrder` response latency remains steady at ~300 ms, well below the 500 ms SLA.
  - **Availability:** External MailFast outages do not impede checkout; orders succeed 100% with messages buffered in queue.
  - **Scalability:** Smoothly absorbs 200 orders/s surges during the 11.11 event without exceeding external rate limits.
  - **Modifiability:** Future requirements (SMS, Fraud Scoring, Recommendations) integrate by binding new consumer queues to `order.v1.events` without touching `checkoutservice`.
- **Negative:**
  - **Operational Overhead:** Requires provisioning, monitoring, and maintaining a high-availability RabbitMQ cluster on Kubernetes.
  - **Eventual Consistency:** Order confirmation emails and loyalty points arrive asynchronously (within seconds to 1 minute) rather than instantaneously.
- **Risks and Mitigations:**
  - *Risk:* Email queue backlog grows excessively during prolonged 200 orders/s peaks.
    *Mitigation:* Configure queue depth alerts, maintain sufficient disk headroom, and schedule automated off-peak backlog draining.
  - *Risk:* RabbitMQ node crash.
    *Mitigation:* Deploy RabbitMQ using Quorum Queues across 3 Kubernetes nodes with persistent volume storage.

## Validation

1. **Load Testing:** Simulate 200 orders/s peak workload for 30 minutes using `loadgenerator`:
   - Validate `PlaceOrder` p95 latency remains ≤ 500 ms.
   - Confirm MailFast outbound request rate never exceeds 50 req/s.
2. **Fault Injection Testing:**
   - Simulate a 20-minute MailFast outage; verify checkout success rate remains unaffected and queued messages drain completely upon recovery.
   - Replay duplicate `order_id` messages to confirm zero duplicate loyalty points are awarded.
3. **SLA Monitoring:** Configure Prometheus/Grafana metrics tracking end-to-end latency from order placement to loyalty point confirmation (p99 < 60 seconds).
