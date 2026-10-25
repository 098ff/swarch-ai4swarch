**Language:** [ภาษาไทย](bonus-recover-decision.md) | English

# Bonus (optional, take-home): Recover an unwritten decision (approximately 15 min)

> **Lecture link:** T5 (Dhar et al. generate decisions from code and diagrams to reduce *knowledge vaporization*), AICH4.
> **Question:** when AI explains *why* a system was built this way, how much of the explanation is evidence and how much is a confident guess?

## Run

Start a new chat or session in your agent and send:

```
Follow prompts/bonus-recover-decision.md
decision: Why do all services communicate synchronously over gRPC, using one shared demo.proto?
evidence: system/online-boutique/protos/demo.proto, system/online-boutique/src/checkoutservice/main.go, system/online-boutique/src/frontend/main.go
```

Want the ADR in Thai? Add a line `language: Thai`.

The agent writes the reconstructed ADR, including its **Claim check** table, to [`outputs/bonus-recovered-adr.md`](outputs/bonus-recovered-adr.md).

## Audit the claim check

Choose **five** claims from the agent's table: at least two it marked *Evidenced* and two it marked *Inferred*. Check each one yourself.

| # | Claim (short) | AI says | You say | Why (file path or reasoning) |
|---|---|---|---|---|
| 1 | `demo.proto` defines request/response RPCs for the 9 listed services | Evidenced | Evidenced | Directly shown in `system/online-boutique/protos/demo.proto` (lines 23–243), which declares unary `rpc ... returns (...)` methods for `CartService` through `AdService` in a single file. |
| 2 | Frontend establishes gRPC client connections to multiple service addresses | Evidenced | Evidenced | Directly shown in `system/online-boutique/src/frontend/main.go` (lines 141–147), where `main()` invokes `mustConnGRPC` to connect `frontendServer` to 7 downstream gRPC services. |
| 3 | Checkout calls downstream RPCs sequentially and uses returned values/errors to place an order | Evidenced | Evidenced | Directly shown in `system/online-boutique/src/checkoutservice/main.go` (`PlaceOrder` lines 230–280 and helpers `prepareOrderItemsAndShippingQuoteFromCart`, `chargeCard`, `shipOrder`), which synchronously await replies and return errors on failure. |
| 4 | The design was selected to provide consistent, discoverable contracts and support result-dependent workflows | Inferred | Inferred | Neither `demo.proto` nor the Go source files contain comments or documentation explaining *why* the authors chose a shared proto or synchronous RPCs; this is a plausible post-hoc rationale inferred from the code structure. |
| 5 | Separate per-service proto definitions could improve independent ownership but make contracts harder to discover | Inferred | Inferred (or Assumption) | The allowed files do not mention rejected alternatives or team ownership boundaries; this is general software architecture trade-off reasoning rather than a fact evidenced in the repository. |

**Count:** of all the agent's claims, how many are *Evidenced* according to the agent? **3 (out of 8)** According to you? **3 (out of 8)**

## Reflect

1. Did the agent ever label a guess as *Evidenced*? Give an example, or explain how you checked that it didn't.
   - **No, the agent did not mislabel any guess as *Evidenced*.** I verified all three claims marked *Evidenced* (Claims #1–#3) by inspecting the exact files and symbols cited: `demo.proto` (lines 23–243) for the 9 service definitions, `frontend/main.go` (lines 141–147) for `mustConnGRPC` client initialization, and `checkoutservice/main.go` (lines 230–386) for synchronous sequential calls in `PlaceOrder`. Every statement about intent, motivation, rejected options, and unmeasured trade-offs (Claims #4–#8) was properly marked *Inferred*.

2. A new developer reads only this reconstructed ADR. What wrong belief could they form?
   - If they overlook the `[Inferred]` tags, they might falsely believe that putting all microservice contracts into a single shared `demo.proto` and chaining synchronous gRPC calls during checkout was a deliberate production best-practice chosen after formal trade-off analysis, rather than a pragmatic simplification for a polyglot demo application. They could also mistakenly assume that *all* inter-service communication in the system uses `demo.proto` over gRPC, missing non-gRPC paths like `frontend` calling `shoppingassistantservice` over HTTP.

3. What can AI recover from code, and what can only the original architects know? Use one example from your ADR.
   - AI can accurately recover the **"What" and "How"** (observable structure, API contracts, and runtime call mechanics)—for example, that `checkoutservice.PlaceOrder` synchronously calls Cart, Product Catalog, Shipping, Currency, Payment, and Email services and uses their return values to build the order response (Claim #3). However, only the original architects know the **"Why"** (historical context, organizational constraints, and true design intent)—for example, whether a single shared `demo.proto` (Claim #4) was chosen to improve contract discoverability, to simplify code generation across multiple languages in a demo repo, or simply due to initial prototyping speed.
