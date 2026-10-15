---
name: review-adr
description: Review an ADR like an architecture board (ATAM-style)
agent: ask
argument-hint: Attach the ADR to review (and the scenario, if it is relevant)
---

Act as a critical architecture reviewer on an architecture review board. Review the attached ADR in the style of ATAM (Architecture Tradeoff Analysis Method).

Produce:

1. **Quality-attribute scenarios.** Two or three concrete scenarios (source · stimulus · environment · response · response measure) that this decision must satisfy. Use numbers from the attached files where they exist.
2. **Findings table** with columns `Type | Finding | Why it matters`, where Type is one of:
   - **Risk**: a decision that may lead to an undesirable outcome
   - **Non-risk**: a decision that is sound, and why
   - **Sensitivity point**: a parameter the quality attribute is highly sensitive to
   - **Trade-off point**: a decision that affects two or more quality attributes in opposite directions
3. **Missing or weak.** Options not considered, consequences not stated, and claims in the ADR that are unsupported or likely wrong.
4. **Verdict.** Accept / accept with changes / reject, with the three most important changes.

Be specific to this system and this ADR. Avoid generic advice that would apply to any system.
