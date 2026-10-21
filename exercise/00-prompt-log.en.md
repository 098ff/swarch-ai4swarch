**Language:** [ภาษาไทย](00-prompt-log.md) | English

# Prompt log

**Name / Student ID:** Chanatda Khonchom / 6631305321 (ชนัดดา คนชม)
**Agent(s) used:** GitHub Copilot (Agent mode in VS Code)
**Model(s):** Auto (gpt-4o / gpt-6-luna)
**Plan / tier:** Copilot Student

Log every request you send to the agent. "Files the agent read" is what the agent reported, or what you saw it open.

| # | Task / run | Agent + model | Message sent (short) | Files the agent read | Stayed within the allowed evidence? | Useful? (1–5) | Notes |
|---|---|---|---|---|---|---|---|
| 1 | 1-A | GitHub Copilot (Agent mode, Auto) | `Follow prompts/1-…` run: A, evidence: `demo.proto` | `AGENTS.md`, `1-recover-architecture.md`, `demo.proto`, `1A-architecture.md` | Yes (read only evidence and prompt files, no source code) | 5 | Extracted 9 service contracts accurately; avoided hallucinating runtime dependencies from proto interface |
| 2 | 1-B | GitHub Copilot (Agent mode, Auto: gpt-6-luna) | `Follow prompts/1-…` run: B, evidence: `demo.proto`, `kubernetes-manifests/` | `AGENTS.md`, `1-recover-architecture.md`, `demo.proto`, `kubernetes-manifests/*.yaml` | Yes (read only demo.proto and k8s manifests as allowed) | 4 | Recovered 12 containers and complete runtime dependencies, but hung at mermaid validation |
| 3 | 2-a | GitHub Copilot (Agent mode, Auto) | `Follow prompts/2-…` run: a, context: none, req: R-12, lang: Thai | `AGENTS.md`, `2-write-adr.md`, `docs/adr/template.md` | Yes (strictly zero-shot: read only template and prompt, did not open scenario.md) | 5 | Structured ADR-0002 well with 3 options; correctly left unknown SLA/numbers to Open Questions without hallucinating |
| 4 | 2-b | GitHub Copilot (Agent mode, Auto) | `Follow prompts/2-…` run: b, context: scenario.md, ADR-0001, checkoutservice/main.go, req: R-12, lang: Thai | `AGENTS.md`, `2-write-adr.md`, `template.md`, `scenario.md`, `0001-…md`, `main.go` | Yes (strictly adhered to allowed context files, prompt, and template) | 5 | Outstanding ADR incorporating exact scenario metrics (p95 500ms, 200 orders/s vs MailFast 50/s), team skills (RabbitMQ), and spotted missing outbox in main.go |
| 5 | 3 | GitHub Copilot (Agent mode, Auto) | `Follow prompts/3-…` adr: 0002-post-order-processing.md, lang: Thai | `AGENTS.md`, `3-review-adr.md`, `0002-post-order-processing.md`, `scenario.md` | Yes (strictly read only the specified ADR, scenario.md, and prompt files) | 5 | Highly incisive ATAM review; critically identified queue bloat math (45 req/s vs 200 orders/s peak), unspecified outbox storage, and rendered 'Accept with changes' |
| 6 | | | | | | | |
| 7 | | | | | | | |
