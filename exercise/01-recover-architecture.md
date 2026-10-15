# Task 1: Recover the architecture (≈ 25 min)

> **Lecture link:** T7 architecture recovery, AICH2 (traceability), AICH3 (context).
> **Question:** how much of an architecture can AI recover, and how much does it depend on the evidence it reads?

For each run: start a **new chat** in **Agent** mode, type `/recover-architecture`, and fill in the inputs. The agent writes its result to `exercise/outputs/1<run>-architecture.md`. Click **Keep** to accept the new file.

| Run | `run` | `evidence` |
|---|---|---|
| A: interfaces only | `A` | `system/online-boutique/protos/demo.proto` |
| B: interfaces + deployment config | `B` | `system/online-boutique/protos/demo.proto, system/online-boutique/kubernetes-manifests/` |
| C: free exploration *(optional; uses more credits)* | `C` | `whole workspace` |

While the agent works, watch which files it opens. If it reads files outside the evidence you allowed, write that down in your prompt log.

## Results

- Run A: [`outputs/1A-architecture.md`](outputs/1A-architecture.md)
- Run B: [`outputs/1B-architecture.md`](outputs/1B-architecture.md)
- Run C (optional): [`outputs/1C-architecture.md`](outputs/1C-architecture.md)

Open each file with **Markdown: Open Preview** (`Ctrl+Shift+V` / `⇧⌘V`) to see the diagram.

## Verify Run B yourself (the important part)

Check **every** dependency in Run B's table against the files. Open the cited file and find the evidence. You may also check the service's source code, e.g. `src/checkoutservice/main.go`.

| Caller → Callee | AI's evidence | Verified? ✅ / ❌ / ❓ | What you found |
|---|---|---|---|
| | | | |
| | | | |
| | | | |

Then list what Run B **got wrong or missed**:

- **False dependencies** (in the diagram but not real): …
- **Missing dependencies** (real but not in the diagram): …
- **Containers to question** (e.g. something present in `src/` but not deployed, or deployed but not in `src/`): …

## Reflect (2–3 sentences each)

1. Compare A and B. What could the agent **not** know from the `.proto` file alone, and why?
2. If you did Run C: did free exploration beat Run B? Which files did the agent choose to read, and was it worth the extra credits?
3. Which **one** finding would mislead a new team member most, if the diagram were published without checking?
4. How does this relate to **AICH2**? Would this diagram stay correct after the next sprint? What would keep it in sync?
