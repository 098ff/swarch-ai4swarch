# Task 2: Recover an unwritten decision (≈ 15 min)

> **Lecture link:** T5 (Dhar et al. generate decisions from code and diagrams to reduce *knowledge vaporization*), AICH4.
> **Question:** when AI explains *why* a system was built this way, how much of the explanation is evidence and how much is a confident guess?

## Run

1. New chat (**Ask** mode). Attach `protos/demo.proto`, `src/checkoutservice/main.go` and `src/frontend/main.go`.
2. Run `/recover-decision` and enter:
   > Why do all services communicate synchronously over gRPC, using one shared `demo.proto`?
3. Paste the full reconstructed ADR, including its **Claim check** table:

<details><summary>AI's reconstructed ADR</summary>

<!-- paste here -->

</details>

## Audit the claim check

Choose **five** claims from the AI's table: at least two it marked *Evidenced* and two it marked *Inferred*. Check each one yourself.

| # | Claim (short) | AI says | You say | Why (file path or reasoning) |
|---|---|---|---|---|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |

**Count:** of all the AI's claims, how many are *Evidenced* according to the AI? ___ According to you? ___

## Reflect

1. Did the AI ever label a guess as *Evidenced*? Give an example, or explain how you checked that it didn't.
2. A new developer reads only this reconstructed ADR. What wrong belief could they form?
3. What can AI recover from code, and what can only the original architects know? Use one example from your ADR.
