**Language:** [ภาษาไทย](05-reflection.md) | English

# Individual reflection (≤ 150 words)

**Name / Student ID:** Chanatda Konchom / 6631305321
**My role on the review board:** AI analyst

After Part A and Part B: which parts of an architect's work would you trust AI with, and which would you keep for humans? Use one concrete example from your board meeting.

Across Parts A and B, I learned that AI excels as a "Chief Analyst" for syntactic and structural tasks, but human architects must remain "Chief Strategists" for organizational and operational governance.

I trust AI to scan codebases, map structural dependencies, draft ADR boilerplate, and identify low-level failure modes. During our board meeting, the AI's most valuable finding was catching the non-transactional Dual-Write risk in `checkoutservice`: a pod crash between charging the credit card and publishing to RabbitMQ drops messages permanently without an Outbox pattern. Human reviewers initially missed this implementation flaw.

However, architectural decisions depend heavily on tacit organizational knowledge, operational constraints, and regulatory compliance that AI cannot infer from code alone. During our board review, the AI recommended self-hosted RabbitMQ based on prompt mentions of developer familiarity. Yet, our SRE revealed that developers had only run RabbitMQ on local laptops—no one had production operational experience, monitoring runbooks did not exist, and cluster budget approvals required two weeks, threatening our three-week 11.11 launch deadline. Furthermore, our Security officer noted pending PDPA legal approvals and strict 30-day data retention limits that the AI could never know.

Ultimately, AI is invaluable for spotting hidden technical edge cases, but human judgment is indispensable for evaluating team capability, business feasibility, compliance, and final risk acceptance.
