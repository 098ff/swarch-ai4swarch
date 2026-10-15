---
name: recover-decision
description: Reconstruct an undocumented architecture decision as an ADR, separating evidence from guesswork
agent: agent
argument-hint: The decision to reconstruct, and the evidence files
---

Nobody wrote an ADR for this decision in Online Boutique:

**${input:decision:Which decision? e.g. why do the services communicate over gRPC with one shared .proto file?}**

**Evidence you may read:** ${input:evidence:e.g. system/online-boutique/protos/demo.proto, system/online-boutique/src/checkoutservice/main.go}

Read only that evidence. Reconstruct the decision as an ADR using [the template](../../docs/adr/template.md), with status **"Reconstructed"**, and write it to `exercise/outputs/2-recovered-adr.md`.

At the end of the file, add a section **Claim check**: a table with one row per claim in the ADR (context, options, rationale, consequences):

| # | Claim | Type | Evidence |
|---|---|---|---|

where **Type** is one of:
- **Evidenced**: the evidence files directly show it (give file path and symbol)
- **Inferred**: plausible, but not shown in the evidence files
- **Assumption**: general industry knowledge, not specific to this system

Be honest. A reconstructed rationale is usually mostly *Inferred*, and that is the point of the exercise.
