---
name: recover-architecture
description: Recover a C4 container diagram of Online Boutique from the attached files
agent: ask
argument-hint: Attach the files you want to use as evidence
---

Recover the **C4 container-level architecture** of the Online Boutique system, using **only the files attached to this chat** as evidence.

Produce, in this order:

1. **Containers table.** One row per container (service or data store): name · technology/language · responsibility · evidence (file path).
2. **Dependencies table.** One row per runtime dependency: caller → callee · protocol · evidence (file path and the symbol or environment variable). Use the most specific evidence you can find.
3. **Mermaid diagram.** A `flowchart LR` C4 container diagram that matches the two tables exactly. Include the end user and any external system.
4. **Gaps.** Bullet points for anything you could not determine from the attached files, and what extra file would settle it.

Do not add containers or dependencies that the attached files don't show. If you think one exists but can't prove it, list it under **Gaps**, not in the diagram.
