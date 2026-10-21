**Language:** [ภาษาไทย](01-recover-architecture.md) | English

# Task 1: Recover the architecture (approximately 15 min)

> **Lecture link:** T7 architecture recovery, AICH2 (traceability), AICH3 (context), and the JSS finding that 93% of studies never validate AI output.
> **Question:** how much of an architecture can AI recover, and how much does it depend on the evidence it reads?

**Don't look at the original Online Boutique repository during this task:** its README shows the architecture diagram you are recovering.

For each run, start a **new chat or session** in your agent and send the message below. The agent follows [`prompts/1-recover-architecture.md`](../prompts/1-recover-architecture.md) and writes its result to `exercise/outputs/1<run>-architecture.md`.

**Run A: interfaces only**
```
Follow prompts/1-recover-architecture.md
run: A
evidence: system/online-boutique/protos/demo.proto
```

**Run B: interfaces + deployment config**
```
Follow prompts/1-recover-architecture.md
run: B
evidence: system/online-boutique/protos/demo.proto, system/online-boutique/kubernetes-manifests/
```

While the agent works, watch which files it opens. If it reads files outside the evidence you allowed, write that down in your prompt log.

## Results

- Run A: [`outputs/1A-architecture.md`](outputs/1A-architecture.md)
- Run B: [`outputs/1B-architecture.md`](outputs/1B-architecture.md)

Open each file with **Markdown: Open Preview** (`Ctrl+Shift+V` / `⇧⌘V`) to see the diagram.

## Verify Run B yourself (the important part)

Check **at least five** dependencies in Run B's table against the files. Open the cited file and find the evidence. You may also check the service's source code, e.g. `src/checkoutservice/main.go`.

| Caller → Callee | AI's evidence | Verified? ✅ / ❌ / ❓ | What you found |
|---|---|:---:|---|
| `loadgenerator` → `frontend` | `system/online-boutique/kubernetes-manifests/loadgenerator.yaml` (`FRONTEND_ADDR`; init container requests `http://${FRONTEND_ADDR}`) | ❓ | The code in `src/loadgenerator/locustfile.py` indeed sends HTTP traffic to frontend using `FRONTEND_ADDR`. However, checking `kubernetes-manifests/kustomization.yaml` shows that `loadgenerator.yaml` is commented out (`# - loadgenerator.yaml`) because it is a synthetic load-testing tool (Locust) used during development, not a production microservice. Placing it inside `System_Boundary` alongside core services distorts the system-level architecture. |
| `frontend` → `cartservice` | `system/online-boutique/kubernetes-manifests/frontend.yaml` (`CART_SERVICE_ADDR`); `system/online-boutique/protos/demo.proto` (`CartService`) | ✅ | Confirmed. `frontend.yaml` configures env `CART_SERVICE_ADDR: "cartservice:7070"`. Code inspection in `src/frontend/main.go` confirms gRPC client initialization (`mustConnGRPC`), and `src/frontend/rpc.go` actively calls the RPC methods (`GetCart`, `InsertCart`, `EmptyCart`) defined in `demo.proto`. |
| `checkoutservice` → `emailservice` | `system/online-boutique/kubernetes-manifests/checkoutservice.yaml` (`EMAIL_SERVICE_ADDR`); `system/online-boutique/protos/demo.proto` (`EmailService`) | ✅ | Confirmed. `checkoutservice.yaml` sets `EMAIL_SERVICE_ADDR: "emailservice:5000"`. Source code in `src/checkoutservice/main.go` shows `sendOrderConfirmation()` calls the gRPC method `SendOrderConfirmation` to send order confirmation emails upon successful checkout. |
| `checkoutservice` → `productcatalogservice` | `system/online-boutique/kubernetes-manifests/checkoutservice.yaml` (`PRODUCT_CATALOG_SERVICE_ADDR`); `system/online-boutique/protos/demo.proto` (`ProductCatalogService`) | ✅ | Confirmed. `checkoutservice.yaml` sets `PRODUCT_CATALOG_SERVICE_ADDR: "productcatalogservice:3550"`. In `src/checkoutservice/main.go`, `prepOrderItems()` invokes the gRPC method `GetProduct` to retrieve product details and prices before computing order totals. |
| `cartservice` → `redis-cart` | `system/online-boutique/kubernetes-manifests/cartservice.yaml` (`REDIS_ADDR`; `redis-cart` Service port named `tcp-redis`) | ✅ | Confirmed. `cartservice.yaml` defines both `cartservice` deployment (with `REDIS_ADDR: "redis-cart:6379"`) and the `redis-cart` service (port 6379, tcp-redis). C# code in `src/cartservice/src/Startup.cs` calls `services.AddStackExchangeRedisCache(...)` over TCP to persist cart session state in Redis (with in-memory fallback if unset). |

- **Wrong or missing** in Run B (false dependencies, missing ones, containers to question):
  1. **Missing Container and Dependency:** `kubernetes-manifests/frontend.yaml` defines `SHOPPING_ASSISTANT_SERVICE_ADDR: "shoppingassistantservice:80"`, and `src/frontend/main.go` executes `mustMapEnv` (frontend crashes on startup if this variable is missing). Furthermore, `src/frontend/handlers.go` dispatches HTTP POST requests to this AI assistant endpoint. However, Run B dropped `shoppingassistantservice` from its containers table, dependencies table, and diagram (relegating it only to Gaps) because no manifest file existed under `kubernetes-manifests/` (it is located under kustomize components). External resources like AlloyDB (vector database) and Gemini API used by this service are also omitted.
  2. **Questionable Container System Boundary:** Placing `loadgenerator` as an internal `Container` within `System_Boundary(shop, "Online Boutique")` alongside core business microservices is an architectural boundary error. It is merely a client-side synthetic load test harness (Locust) and is explicitly commented out in `kustomization.yaml` for production.
  3. **Missing Implementation Language & Tech Stack:** Run B only recovered technology as generic container image names and protocols ("language not evidenced"), failing to identify actual implementation languages (e.g., Go, C#, Java, Python, Node.js) because K8s manifests only contain image tags rather than runtime code metadata.
  4. **Unrevealed External Boundaries (Mock vs External Gateway):** `paymentservice` and `emailservice` are represented as self-contained services, whereas code inspection shows they are simplistic mock implementations with no actual integration to third-party payment gateways or SMTP servers.

## Reflect (2–3 sentences each)

1. Compare A and B. What could the agent **not** know from the `.proto` file alone, and why?
   - The agent cannot determine **Caller → Callee runtime dependencies (call graph or system topology)**, actual deployment topology (such as separate containers or data stores like `redis-cart`), or implementation technologies and languages.
   - This is because a `.proto` file is strictly an Interface Definition Language (IDL) serving as a service contract that specifies "what capabilities are offered" (RPC methods and data types), without any information on "who invokes whom" or how services are packaged and executed at runtime (aligning with challenge **AICH3: Context, not just structure** in Architecture Recovery T7, where AI captures syntax/specifications but misses runtime semantics and architectural context).

2. Which **one** mistake would mislead a new team member most if the diagram were published without checking?
   - The most misleading mistake is **placing `loadgenerator` as an internal `Container` inside the `System_Boundary` alongside production microservices**.
   - A new team member would mistakenly believe that `loadgenerator` is a core business service required to be deployed, scaled, and monitored in production, whereas it is actually an external test harness (Locust) used only during development and commented out in `kustomization.yaml`.
   - In the C4 model, mixing development test harnesses into the core system boundary misrepresents production scope, distorts non-functional requirement analysis (such as security policies and capacity planning), and obscures the true customer journey.
