# Task 3: Write an ADR for a new requirement (≈ 20 min)

> **Lecture link:** T3 (GPT-4 zero-shot was coherent but *less complete than humans*; few-shot closed much of the gap), AICH3 (context), W10 (message brokers).
> **Question:** how much does an AI-written ADR improve when you give it an example and the real context?

Read [`docs/scenario.md`](../docs/scenario.md) first: requirement **R-12**, what happens after an order is placed.

For both runs: start a **new chat** in **Agent** mode, type `/write-adr`, and paste the requirement text (the quoted R-12 block from the scenario).

| Run | `run` | `context` | Output |
|---|---|---|---|
| a: zero-shot | `a` | `none` | [`outputs/3a-adr.md`](outputs/3a-adr.md) |
| b: example + context | `b` | `docs/scenario.md, docs/adr/0001-use-redis-for-cart-storage.md, system/online-boutique/src/checkoutservice/main.go` | [`outputs/3b-adr.md`](outputs/3b-adr.md) |

> In Run a, check that the agent really read nothing besides the template. If it opened `docs/scenario.md` anyway, note that in your prompt log, because it changes the comparison.

## Compare a and b

| Criterion | Run a | Run b |
|---|---|---|
| Uses the scenario's numbers (latency, peak, MailFast limits)? | | |
| Options considered (list them) | | |
| Does the chosen option meet *every* quality goal in the scenario? | | |
| Handles "exactly once" loyalty points? How? | | |
| Handles MailFast's rate limit and slowness? How? | | |
| Anything invented (a number, tool or fact not in the scenario)? | | |
| Open questions it raised | | |

## Take ownership: your final ADR

You are the **Chief Strategist**. Copy `outputs/3b-adr.md` to `docs/adr/0002-<short-title>.md`, and edit it **yourself** until you would sign it. Don't ask the agent to do this part.

- Fix anything wrong or invented.
- Add what is missing.
- Make the final call yourself. You may disagree with the AI.

Then summarise your edits:

| # | What you changed | Why |
|---|---|---|
| 1 | | |
| 2 | | |
| 3 | | |
