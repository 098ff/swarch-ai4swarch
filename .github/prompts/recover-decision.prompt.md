---
name: recover-decision
description: Reconstruct an undocumented architecture decision as an ADR, separating evidence from guesswork
agent: ask
argument-hint: The decision to reconstruct, e.g. "why the services talk over gRPC"
---

Nobody wrote an ADR for this decision in Online Boutique:

**${input:decision:Which decision? e.g. why do the services communicate over gRPC with one shared .proto file?}**

Reconstruct it as an ADR using [the template](../../docs/adr/template.md), with status **"Reconstructed"**. Use the files attached to this chat as evidence.

Then add a section at the end, **Claim check**, as a table with one row per claim in the ADR (context, options, rationale, consequences):

| # | Claim | Type | Evidence |
|---|---|---|---|

where **Type** is one of:
- **Evidenced**: the attached code or config directly shows it (give file path and symbol)
- **Inferred**: plausible, but not shown in the attached files
- **Assumption**: general industry knowledge, not specific to this system

Be honest. A reconstructed rationale is usually mostly *Inferred*, and that is the point of the exercise.
