# Task 1 prompt: Recover the architecture

## Inputs (from the student's message)

- `run`: `A`, `B` or `C`
- `evidence`: the files and folders you may read, or `whole workspace`

## Instructions

Recover the **C4 container-level architecture** of the Online Boutique system.

Read **only** the `evidence`. If it is `whole workspace`, explore `system/online-boutique/` as you see fit, and list every file you opened.

Write the result to `exercise/outputs/1<run>-architecture.md` (e.g. `exercise/outputs/1A-architecture.md`), with these sections in order:

1. **Agent.** Your product name and model, if you know them.
2. **Files read.** The exact files you opened.
3. **Containers table.** One row per container (service or data store): name · technology/language · responsibility · evidence (file path).
4. **Dependencies table.** One row per runtime dependency: caller → callee · protocol · evidence (file path and the symbol or environment variable). Use the most specific evidence you can find.
5. **Mermaid diagram.** A `flowchart LR` C4 container diagram that matches the two tables exactly. Include the end user and any external system.
6. **Gaps.** Anything you could not determine from the evidence, and which extra file would settle it.

Do not add containers or dependencies that the evidence doesn't show. If you think one exists but can't prove it, list it under **Gaps**, not in the diagram.
