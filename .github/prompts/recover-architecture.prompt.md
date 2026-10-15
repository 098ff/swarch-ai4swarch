---
name: recover-architecture
description: Recover a C4 container diagram of Online Boutique from the evidence you allow
agent: agent
argument-hint: Run letter (A, B or C) and the evidence files/folders
---

Recover the **C4 container-level architecture** of the Online Boutique system.

- **Run:** ${input:run:A, B or C}
- **Evidence you may read:** ${input:evidence:e.g. system/online-boutique/protos/demo.proto — or "whole workspace" for run C}

Read **only** the evidence listed above. If it says "whole workspace", explore `system/online-boutique/` as you see fit, and list every file you opened.

Write the result to `exercise/outputs/1${input:run}-architecture.md`, with these sections in order:

1. **Files read.** The exact files you opened.
2. **Containers table.** One row per container (service or data store): name · technology/language · responsibility · evidence (file path).
3. **Dependencies table.** One row per runtime dependency: caller → callee · protocol · evidence (file path and the symbol or environment variable). Use the most specific evidence you can find.
4. **Mermaid diagram.** A `flowchart LR` C4 container diagram that matches the two tables exactly. Include the end user and any external system.
5. **Gaps.** Anything you could not determine from the evidence, and which extra file would settle it.

Do not add containers or dependencies that the evidence doesn't show. If you think one exists but can't prove it, list it under **Gaps**, not in the diagram.
