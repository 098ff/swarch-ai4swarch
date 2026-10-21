Reviewer: AI assistant using Copilot SDK in VS Code; Model: not disclosed

## Quality-attribute scenarios

1. **Latency / Availability** — Source: Customers · Stimulus: Invoking `PlaceOrder` during peak load · Environment: 200 orders/s peak load while MailFast is slow (p95 1.5 s, p99 8 s) or unavailable · Response: Acknowledge the order placement without waiting for post-order consumers · Response measure: `PlaceOrder` p95 latency ≤ 500 ms and MailFast outages do not cause order failures (`docs/scenario.md`, ADR-0002: Decision/Validation).
2. **Correctness / Availability** — Source: Ordering customer · Stimulus: `OrderPlaced` event is redelivered or `loyaltyservice` crashes · Environment: 200 orders/s peak load with up to 3 retries before routing to DLQ · Response: Loyalty points are awarded exactly once and processed within SLA · Response measure: Points awarded within 1 minute and no duplicate awards (`docs/scenario.md`, ADR-0002: Decision/Retry Policy).
3. **Scalability / Modifiability** — Source: Marketing team · Stimulus: Adding new post-order actions such as SMS · Environment: 200 orders/s peak load with MailFast rate limited to 50 emails/s · Response: Add independent consumers without modifying `checkoutservice` and without unbounded queue growth · Response measure: Consumer consumption rate balances incoming rate and downstream constraints (`docs/scenario.md`; ADR-0002: Decision/Consequences).

## Findings

| Type | Finding | Why it matters |
|---|---|---|
| Risk | Email consumer is throttled at 45 requests/s, but peak load reaches 200 orders/s; assuming 1 order triggers 1 email, ingress exceeds egress by at least 155 msgs/s | The email queue will grow unbounded during sustained peak periods even when MailFast operates normally. A queue buffer cannot compensate for an absolute deficit in downstream processing capacity (`docs/scenario.md`: Peak load/MailFast; ADR-0002: Rate Limiting). |
| Risk | ADR claims loyalty points are awarded within 1 minute with an "Exactly-once outcome", but lacks explicit consumer throughput targets, retry delays, or DLQ recovery procedures; messages exceeding retries into DLQ risk missing the SLA or losing points | The correctness and timeliness goals lack proof of viability under 200 orders/s peak load and the strict 1-minute window (`docs/scenario.md`; ADR-0002: Retry Policy/Idempotency). |
| Sensitivity point | Loyalty consumer replica count, throughput per replica, peak duration, retry backoff, and queue depth directly dictate whether the 1-minute SLA is met | Merely setting alerts on high queue depth detects lag but does not guarantee sufficient processing power or automated backlog remediation (ADR-0002: Risks and Mitigations). |
| Risk | Transactional Outbox is stated to write order state and events within a single transaction, but the ADR does not specify the actual persistent store or transaction boundary; current codebase lacks verified atomic storage support | Without atomicity between order persistence and outbox event recording, message loss or duplicate publishing can occur; current system shows only in-memory flow without evidenced database support (`docs/scenario.md`: How it works today; ADR-0002: Transactional Outbox). |
| Trade-off point | Asynchronous decoupling shields `PlaceOrder` response times from MailFast, loyalty, and analytics latency, but accepts eventual consistency without defining maximum completion time for emails and analytics | Protecting checkout latency/availability comes at the expense of post-order customer experience immediacy; only loyalty has a specified 1-minute SLA (`docs/scenario.md`; ADR-0002: Consequences). |
| Non-risk | Independent queues per consumer isolate failures and allow adding new actions without introducing synchronous dependencies into checkout | Aligns directly with requirements that post-order downstream failures must not fail order checkout, and accommodates future extensibility (`docs/scenario.md`: R-12/Availability; ADR-0002: RabbitMQ Topology). |

## Missing or weak

- **Unconsidered options:** Asynchronous messaging without Outbox pattern, or alternative broker technologies self-hostable by the team; rationale for RabbitMQ relative to alternatives remains sparse.
- **Unstated consequences:** Quantitative queue accumulation rates, behavior during disk exhaustion, DLQ manual/automated replay procedures, message ordering, and broker failover recovery.
- **Unsupported claims:** Phrases like "constant ~300 ms", "100% success rate", and handling 200 orders/s "comfortably" lack empirical load testing or formal capacity modeling; proposed tests are plans, not verified results.
- **Exactly-once:** Unique constraints on idempotency keys prevent duplicate awards only when key verification and balance updates are atomic; the ADR should clarify this precondition rather than claiming duplicate awards are "impossible".

## Verdict

**Accept with changes.** Prior to finalizing the ADR:
1. Address the MailFast capacity deficit under 200 orders/s peak ingress by defining acceptable queue backlog limits and an off-peak draining strategy; throttling alone to 45/s is mathematically insufficient.
2. Define explicit throughput/latency targets for loyalty processing, along with retry policies, DLQ replay procedures, and atomic idempotency guarantees.
3. Specify the persistent datastore and transaction boundary for the Outbox pattern, including publisher ack semantics, and frame latency/availability benefits as testable targets rather than guaranteed certainties.
