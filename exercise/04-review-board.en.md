**Language:** [ภาษาไทย](04-review-board.md) | English

# Part B: Architecture review board (approximately 25 min, groups of 4)

> **Lecture link:** AICH4 (reviews need domain expertise; *plausible ≠ appropriate*), Pillar 5 (AI as Chief Analyst, humans as Chief Strategist), JSS (85% of studies keep a human in the loop).
> **Question:** what does a human review board catch that the AI's review missed, and why?

The company's architecture review board meets to approve the R-12 design before the 11.11 sale. One ADR goes to the board: the **architect** presents it, the **AI analyst** presents the AI's review, and two stakeholders bring knowledge the AI never saw.

## 1. Form the board (2 min)

In groups of 4, choose **one** member's ADR to review (`docs/adr/0002-*.md`, plus their `exercise/outputs/3-review.md`). Then assign roles:

| Role | Who | Brings |
|---|---|---|
| **Architect** | Naphat Serirak (6632061321) | defends the ADR |
| **AI analyst** | Chanatda Konchom (6631305321) | reads out the AI's review only, and may not add their own opinions |
| **Product owner** | Ittichet Thongsang (6631363721) | role card P (business side) |
| **Site reliability engineer (SRE)** | Chayut Archamongkol (6631307621) | role card S (operations side) |
| **Security & privacy officer** | Napat Srisamut (6631314021) | role card X (security & privacy) |

Groups of 5 add a **security & privacy officer** (role card X). In groups of 3, the architect also plays the AI analyst.

**Role cards** are in [`docs/review-board/`](../docs/review-board/README.en.md). Each stakeholder reads **only their own card** and keeps it secret until the meeting.

## 2. Meeting (15 min)

| Min | Step |
|---|---|
| 3 | **Architect** presents the decision, the options considered and the main consequences |
| 3 | **AI analyst** reads the AI review's findings and verdict, word for word |
| 7 | **Open discussion.** Each stakeholder raises the concerns from their card, in their own words. The architect responds; anyone may ask the AI analyst "did the AI see this?" |
| 2 | **Decision.** The board votes: accept / accept with changes / reject |

## 3. Record the decision (8 min, together)

**ADR reviewed:** `docs/adr/0002-use-rabbitmq-for-post-order-processing.md` by Naphat Serirak (6632061321) ([GitHub Link](https://github.com/neennera/naphat-salab-aiagent/blob/main/docs/adr/0002-use-rabbitmq-for-post-order-processing.md))
**Board members and roles:**
- **Architect:** Naphat Serirak (6632061321)
- **AI analyst:** Chanatda Konchom (6631305321)
- **Product owner:** Ittichet Thongsang (6631363721)
- **Site reliability engineer (SRE):** Chayut Archamongkol (6631307621)
- **Security & privacy officer:** Napat Srisamut (6631314021)

**Decision:** accept with changes

**Required changes:**
1. **Mitigate Dual-Write Risk (from AI review):** Implement a Transactional Outbox pattern or local persistent write in `checkoutservice` to guarantee that paid orders are never lost if the pod crashes before publishing to RabbitMQ.
2. …
3. …
4. …
5. …

**Who raised what?** List every concern the board discussed.

| # | Concern | Raised by (AI review / architect / PO / SRE / security) | Was it in the AI's review? | Could the AI have known? Why or why not? |
|---|---|---|---|---|
| 1 | **Dual-Write / Crash-after-Payment Risk:** If pod restarts right after credit card charge but before publishing to RabbitMQ, message is lost. | AI review | Yes | Yes; the AI analyzed `checkoutservice/main.go` and recognized two decoupled sequential calls without transactional guarantees. |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |

**The AI's best finding** (one the humans had missed): The identification of the Dual-Write risk (crash-after-payment) in `checkoutservice`, exposing the critical flaw where customer cards are charged but RabbitMQ publication could fail silently without an Outbox pattern.
**The AI's weakest finding** (generic, wrong or irrelevant): The generic warning about Eventual Consistency impacting UX, which provided standard textbook commentary without offering concrete business mitigations or operational thresholds.

## 4. Individual reflection (≤ 150 words, each student, after class is fine)

Write in [`05-reflection.md`](05-reflection.en.md): after Part A and Part B, which parts of an architect's work would you trust AI with, and which would you keep for humans? Use one concrete example from your board meeting.

> Each group member commits this record to their own repository (copy the same text), so everyone submits a complete repo.
