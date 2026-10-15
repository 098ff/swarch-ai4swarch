# Task 1: Recover the architecture (≈ 20 min)

> **Lecture link:** T7 architecture recovery, AICH2 (traceability), AICH3 (context).
> **Question:** how much of an architecture can AI recover, and how much does it depend on the context you give it?

## Run A: interfaces only

1. New chat (**Ask** mode). Attach only `system/online-boutique/protos/demo.proto`.
2. Run `/recover-architecture`.
3. Paste the **Mermaid diagram** below.

```mermaid
%% paste Run A diagram here
```

## Run B: interfaces + deployment config

1. New chat. Attach `protos/demo.proto` **and** the whole `system/online-boutique/kubernetes-manifests/` folder.
2. Run `/recover-architecture` again.
3. Paste the **Mermaid diagram** and the **dependencies table** below.

```mermaid
%% paste Run B diagram here
```

<details><summary>Run B dependencies table</summary>

<!-- paste here -->

</details>

## Verify Run B yourself (the important part)

Check **every** dependency in Run B against the files. Open the cited file and find the evidence. You may also check the service's source code, e.g. `src/checkoutservice/main.go`.

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

1. Compare A and B. What could the AI **not** know from the `.proto` file alone, and why?
2. Which **one** finding would mislead a new team member most, if the diagram were published without checking?
3. How does this relate to **AICH2**? Would this diagram stay correct after the next sprint? What would keep it in sync?
