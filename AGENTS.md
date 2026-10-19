# Rules for AI agents in this repository

You are assisting software architecture students. They are analysing the **Online Boutique** microservices system in `system/online-boutique/` and writing architecture documentation about it.

These rules apply to **every** AI agent used in this repository (GitHub Copilot, Gemini, Claude, Codex, Cursor, Cline, …). Follow them for every task.

## How tasks work

The student sends you a message like:

```
Follow prompts/1-recover-architecture.md
run: A
evidence: system/online-boutique/protos/demo.proto
```

1. Open the named file in `prompts/` and follow its instructions exactly.
2. Take the input values (`run`, `evidence`, …) from the student's message. If a required input is missing, ask for it before doing anything else.

## Scope of what you read and change

- **Read only what the task allows.** Besides the task file itself, each task names the files or folders you may use as evidence. Do not open or search other files. Comparing what you can conclude from different evidence is part of the exercise.
- **Never read `docs/review-board/`.** Those role cards are for the human review board only.
- **Write only the output file named in the task.** Create it if it doesn't exist. Never modify anything under `system/`, `prompts/` or `docs/adr/0001-*`, never edit this file, and never edit the student's worksheets in `exercise/*.md`.
- **Don't run terminal commands.** Everything in this exercise can be done by reading files.
- When you finish, reply with a short summary: what you wrote, where, and **every file you read**.

## How to answer

- **Evidence first.** Support every architectural claim (a service, a dependency, a protocol, a data store, a rationale) with the file path, and the line or symbol where possible, that shows it. Example: `system/online-boutique/kubernetes-manifests/frontend.yaml (CART_SERVICE_ADDR)`.
- **Say when you don't know.** If the allowed files don't show something, write "Not evidenced in the provided files". Don't fill gaps with what such systems usually do. If you offer a plausible explanation that the code doesn't support, label it **[Inferred]**.
- **Be concise.** Prefer tables and short bullet points over long prose.

## Diagrams

- Write diagrams in Mermaid inside a ```` ```mermaid ```` code block.
- For C4 container diagrams, use `flowchart LR`:
  - one node per container (service or data store), labelled `name<br/>[technology]`
  - every edge labelled with its protocol, e.g. `-->|gRPC|`
- Keep node IDs simple (letters and digits only) so the diagram renders.

## Architecture decision records

- ADRs follow the template in `docs/adr/template.md`. Keep all of its headings.
- List at least two genuinely different options, each with real pros and cons.
- Link each consequence to a quality attribute (performance, availability, modifiability, cost, …).
