# Task 4: Review it like an architecture board (≈ 15 min)

> **Lecture link:** AICH4 (plausible ≠ appropriate), P5 human–AI collaboration (AI as Chief Analyst, you as Chief Strategist). A 2026 study found LLMs often matched or beat student teams at spotting ATAM risks and trade-offs ([arXiv 2603.28914](https://arxiv.org/abs/2603.28914)). Let's test that claim.

## Run

Start a new chat or session and send, with the path of **your final ADR**:

```
Follow prompts/4-review-adr.md
adr: docs/adr/0002-<your-title>.md
```

The agent writes its review to [`outputs/4-review.md`](outputs/4-review.md). It must not edit your ADR. If it does, reject or undo the change.

## Judge the reviewer

For each of the agent's findings, decide whether you agree.

| # | AI finding (short) | Type (risk / sensitivity / trade-off / …) | Agree? | Your reasoning |
|---|---|---|---|---|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |

- **One finding you had missed, but is right:** …
- **One finding that is generic or wrong:** …
- **Did you change your ADR after the review?** What, and why?

## Score the three ADRs

Score each criterion from 0 to 2: 0 = missing or wrong, 1 = partly, 2 = good.

| Criterion | 3a zero-shot | 3b with context | 0002 your final |
|---|---|---|---|
| Context: problem and real constraints stated | | | |
| Options: ≥ 2 genuinely different, fair pros and cons | | | |
| Fit: the decision meets the scenario's quality goals | | | |
| Consequences: tied to quality attributes, incl. negatives | | | |
| Honesty: nothing invented; open questions flagged | | | |
| **Total (/10)** | | | |

## Final reflection (≤ 150 words)

Which parts of an architect's work in this exercise would you now trust an AI agent with, and which would you keep for yourself? Use evidence from Tasks 1–4.
