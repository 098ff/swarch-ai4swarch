# Task 1: Recover the architecture (≈ 25 min)

> **Lecture link:** T7 architecture recovery, AICH2 (traceability), AICH3 (context).
> **Question:** how much of an architecture can AI recover, and how much does it depend on the evidence it reads?

For each run, start a **new chat or session** in your agent and send the message below. The agent follows [`prompts/1-recover-architecture.md`](../prompts/1-recover-architecture.md) and writes its result to `exercise/outputs/1<run>-architecture.md`.

**Run A: interfaces only**
```
Follow prompts/1-recover-architecture.md
run: A
evidence: system/online-boutique/protos/demo.proto
```

**Run B: interfaces + deployment config**
```
Follow prompts/1-recover-architecture.md
run: B
evidence: system/online-boutique/protos/demo.proto, system/online-boutique/kubernetes-manifests/
```

**Run C: free exploration** *(optional; uses the most credits)*
```
Follow prompts/1-recover-architecture.md
run: C
evidence: whole workspace
```

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

## Compare with a classmate who used a different agent

Find someone who used a different agent or model. Compare your Run B results.

| | Your agent: … | Their agent: … |
|---|---|---|
| Containers found | | |
| Dependencies found | | |
| False or missing dependencies | | |

## Reflect (2–3 sentences each)

1. Compare A and B. What could the agent **not** know from the `.proto` file alone, and why?
2. If you did Run C: did free exploration beat Run B? Which files did the agent choose to read, and was it worth the extra cost?
3. Two agents gave different diagrams of the same code. What does that tell you about using any single AI output as documentation?
4. How does this relate to **AICH2**? Would this diagram stay correct after the next sprint? What would keep it in sync?
