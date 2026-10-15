---
name: write-adr
description: Draft an ADR for a new requirement on Online Boutique
agent: agent
argument-hint: Run (a or b), the context files (or "none"), and the requirement
---

Draft an architecture decision record for this new requirement on Online Boutique:

**${input:requirement:Paste the requirement here}**

- **Run:** ${input:run:a (zero-shot) or b (with context)}
- **Context you may read:** ${input:context:"none", or e.g. docs/scenario.md, docs/adr/0001-use-redis-for-cart-storage.md, system/online-boutique/src/checkoutservice/main.go}

Apart from [the ADR template](../../docs/adr/template.md), read **only** the context listed above. If it says "none", read nothing else, and write from the requirement alone.

Write the ADR to `exercise/outputs/3${input:run}-adr.md`. Keep every template heading, set status to **"Proposed"**, and number it 0002.

- If an example ADR or scenario is in your context, follow its style and use its facts (quality goals, constraints, numbers).
- If something you need is not given (a number, a constraint, a team skill), don't invent it. List it under **Open questions** at the end.
