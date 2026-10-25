**Language:** [ภาษาไทย](05-reflection.md) | English

# Individual reflection (≤ 150 words)

**Name / Student ID:** Chanatda Konchom / 6631305321
**My role on the review board:** AI analyst

After Part A and Part B: which parts of an architect's work would you trust AI with, and which would you keep for humans? Use one concrete example from your board meeting.

I trust AI as a "Chief Analyst" for technical and structural tasks: scanning codebases to recover static dependencies, drafting initial ADRs, and uncovering subtle code-level architectural flaws. In our board meeting, the AI's best finding was detecting the non-transactional Dual-Write risk in `checkoutservice` (crash-after-payment), which the human team had overlooked.

However, I would keep governance, operational viability, and business trade-offs strictly for humans. AI cannot reason about tacit organizational knowledge, regulatory status, or team capabilities outside its prompt context. For instance, while the AI suggested RabbitMQ, our SRE revealed that developers only ran it on laptops with zero production experience, and cluster budget approvals would delay the 11.11 launch. Humans must remain Chief Strategists.
