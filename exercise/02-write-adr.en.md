**Language:** [ภาษาไทย](02-write-adr.md) | English

# Task 2: Write an ADR for a new requirement (approximately 20 min)

> **Lecture link:** T3 (GPT-4 zero-shot was coherent but *less complete than humans*; few-shot closed much of the gap), AICH3 (context), W10 (message brokers).
> **Question:** how much does an AI-written ADR improve when you give it an example and the real context?

Read [`docs/scenario.md`](../docs/scenario.md) first: requirement **R-12**, what happens after an order is placed.

For each run, start a **new chat or session** and send the message below. Replace `<paste R-12 here>` with the quoted R-12 block from the scenario.

**Run a: zero-shot** → [`outputs/2a-adr.md`](outputs/2a-adr.md)
```
Follow prompts/2-write-adr.md
run: a
context: none
requirement: <paste R-12 here>
```

**Run b: example + context** → [`outputs/2b-adr.md`](outputs/2b-adr.md)
```
Follow prompts/2-write-adr.md
run: b
context: docs/scenario.md, docs/adr/0001-use-redis-for-cart-storage.md, system/online-boutique/src/checkoutservice/main.go
requirement: <paste R-12 here>
```

> Want the ADR in Thai? Add a line `language: Thai` to both messages (template headings and technical names stay in English). **Use the same language in Run a and Run b**, so the comparison is fair.

> In Run a, check that the agent really read nothing besides the template. If it opened `docs/scenario.md` anyway, note that in your prompt log, because it changes the comparison.

## Compare a and b

| Criterion | Run a | Run b |
|---|---|---|
| Uses the scenario's numbers (latency, peak, MailFast limits)? | **Did not use any numbers.** Explicitly noted that there was no data on protocols, workload, latency targets, SLA, or messaging mechanism, and relegated them to Open Questions. | **Used accurately and comprehensively.** Directly cited checkout p95 latency budget ≤ 500 ms (current ~300 ms), 11.11 peak load of 200 orders/s, MailFast characteristics (p95 1.5s, p99 8s, failure 0.5%, rate limit 50 emails/s), and loyalty SLA within 1 minute. |
| Options considered (list them) | 1. Direct synchronous calls to external endpoints<br>2. Generic durable event-driven architecture with decoupled consumers<br>3. Centralized post-order orchestrator | 1. Direct synchronous calls from `checkoutservice`<br>2. Publish post-order events via **RabbitMQ** with dedicated consumers (tailored to team skills)<br>3. Synchronous post-order orchestration service |
| Handles "exactly once" loyalty points and MailFast's rate limit? How? | **Addressed only conceptually.** Generically stated that consumers must be idempotent to avoid duplicate points, and deferred MailFast rate limits to an Open Question. | **Handled concretely.** Proposed using the existing `orderID` from `PlaceOrder` as an idempotency key for deduplication in the loyalty consumer, and utilized RabbitMQ queues to buffer 200 orders/s peak bursts while throttling email dispatch below 50 emails/s. |
| Anything invented (a number, tool or fact not in the scenario)? | **No fabricated facts, but made generic assumptions.** Because context was absent, it proposed abstract solutions without grounding in existing system constraints. | **No hallucinations.** Faithfully grounded all trade-offs in facts from `scenario.md` and code in `main.go`. Accurately identified that `main.go` currently lacks a transactional outbox and persistent order storage. |

## Take ownership: your final ADR

You are the architect. Copy `outputs/2b-adr.md` to `docs/adr/0002-<short-title>.md` and edit it **yourself** until you would sign it. Don't ask the agent to do this part.

- Fix anything wrong or invented, and add what is missing.
- Make the final call yourself. You may disagree with the AI.

You will defend this ADR in front of a review board in Part B, so make sure you can explain every decision in it.

| # | What you changed from the AI's draft | Why |
|---|---|---|
| 1 | **Added Transactional Outbox Pattern and persistent storage to `checkoutservice` prior to event publishing** | AI's draft 2b only noted the lack of an outbox and order store as an open question. As the architect, I formally decided to introduce the Outbox Pattern to guarantee data durability and at-least-once delivery, preventing lost post-order events if RabbitMQ is temporarily down. |
| 2 | **Specified concrete RabbitMQ topology: Fanout Exchange (`order.v1.events`), 3 independent queues, and Dead-Letter Queues (DLQ) with retry** | AI proposed RabbitMQ at a high conceptual level. The architect concretized this with a fanout exchange and 3 decoupled queues for fault isolation, adding DLQs with exponential backoff so external downtime (MailFast) does not poison or block main queue consumers. |
| 3 | **Implemented Token Bucket Rate Limiting (capped at 45 req/s) and a Circuit Breaker on the Email Consumer** | AI only mentioned "limiting concurrency" vaguely. The architect enforced an explicit 45 req/s throttle (providing a safety buffer below MailFast's 50 req/s hard limit) and added a circuit breaker to prevent cascading failures during 200 orders/s peak surges. |
