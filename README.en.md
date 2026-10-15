**Language:** [ภาษาไทย](README.md) | English

# AI for Software Architecture: Hands-on Exercise

**2110521 Software Architecture · Week 12 · Tutorial 5 / Assignment 5**

You'll use **an AI coding agent in VS Code** as an architecture assistant on a real microservices system, [Online Boutique](system/online-boutique/). Use whichever agent you have: GitHub Copilot, Gemini, Claude, Codex, Cursor or another. Then you'll judge how good its work is. The goal is not to get the AI to do the assignment. The goal is to find out **where AI helps an architect and where it can't be trusted**, which is the main finding of this week's lecture.

You will:

1. **Recover** the architecture from the code (lecture topic T7: architecture recovery)
2. **Recover a decision** that was never written down (T5: decision recovery, knowledge vaporization)
3. **Make a new decision** with an AI-written ADR, with and without context (T3: ADR generation)
4. **Review** the ADR the way an architecture board would, then take responsibility for the final version (AICH4, "Chief Strategist")

Expected time: about 75 minutes in class.

---

## Before class: set up (≈ 15 minutes, do it early)

1. **Install [VS Code](https://code.visualstudio.com/).**
2. **Get an AI agent that can read and write files in your workspace.** Any of these works:

   | Agent | Where it runs in VS Code | Free option for students |
   |---|---|---|
   | **GitHub Copilot**, in **Agent** mode | Chat view | Copilot Student via [GitHub Education](https://docs.github.com/en/copilot/how-tos/copilot-on-github/set-up-copilot/enable-copilot/set-up-for-students) (approval can take days), or Copilot Free |
   | **Gemini Code Assist**, in agent mode | Gemini panel | Free tier with a personal Google account |
   | **Gemini CLI** | VS Code terminal | Free tier with a Google account |
   | **Claude Code**, **Codex**, **Cursor**, **Cline**, … | Extension, terminal or editor | Depends on the tool |

   Plain chat that can't open files won't work, because the agent must read the code and write its results to files.
3. **Create your own copy of this repository.**
   - Click **Use this template → Create a new repository**. It can be private.
   - Clone it and open the folder in VS Code.
4. **Install the recommended extension** when VS Code asks: **Markdown Preview Mermaid Support**, which renders the diagrams you will produce.
5. **Check that your agent can see the rules.** Open a new chat or session and ask: *"What are the rules for AI agents in this repository?"* It should summarise [`AGENTS.md`](AGENTS.md). If it doesn't, start each task message with `Read AGENTS.md first.`

## How the exercise works with any agent

- **The rules are in [`AGENTS.md`](AGENTS.md).** Most agents read it automatically. Small pointer files load the same rules for tools that look elsewhere: `GEMINI.md` for Gemini, `CLAUDE.md` for Claude Code, `.github/copilot-instructions.md` for Copilot.
- **The tasks are plain Markdown files in [`prompts/`](prompts/).** The worksheets give you the exact message to send, for example:

  ```
  Follow prompts/1-recover-architecture.md
  run: A
  evidence: system/online-boutique/protos/demo.proto
  ```

- **The agent writes its result to `exercise/outputs/`.** You review that file, then accept the change. Reject or undo anything else it tries to change.
- **The agent must not** edit `system/`, `prompts/`, the example ADR or your worksheets, and must not run terminal commands. If it asks to run a command, decline.

Because everyone uses the same task files and rules, results from different agents can be compared. Comparing them is part of Task 1.

## Budget

The core tasks need about **8–10 agent requests**. Optional Run C in Task 1 adds one more, but it's an expensive one.

- Each agent request reads files and writes a result, so it uses more of your quota than a chat answer. **Reading less is cheaper.**
- **Start a new chat or session for every run.** Old conversations are sent again with each request.
- If your free quota runs out, you can switch to another agent mid-exercise. Note it in your prompt log.

## Rules of the game

- **Log every request** in [`exercise/00-prompt-log.md`](exercise/00-prompt-log.md): which agent and model, what you asked, which files the agent read, and whether it stayed within the evidence you allowed.
- **Check claims against the code.** A claim without a file path as evidence is a guess until you verify it.
- **Never paste AI output as your own conclusion** without marking it. The worksheets have separate places for *the AI's answer* and *your judgement*.

## The tasks

| # | Worksheet | Task file | Agent requests |
|---|---|---|---|
| 1 | [Recover the architecture](exercise/01-recover-architecture.md) | `prompts/1-recover-architecture.md` | 2 (+1 optional) |
| 2 | [Recover an unwritten decision](exercise/02-recover-decision.md) | `prompts/2-recover-decision.md` | 1 |
| 3 | [Write an ADR for a new requirement](exercise/03-write-adr.md) | `prompts/3-write-adr.md` | 2 |
| 4 | [Review it and take ownership](exercise/04-review-adr.md) | `prompts/4-review-adr.md` | 1 |

> The worksheets are in English. You may answer in Thai or English.

## Submission (Assignment 5, due by the end of class)

1. Fill in all worksheets in `exercise/`. Keep the agent's files in `exercise/outputs/`, and write your final ADR at `docs/adr/0002-*.md`.
2. Commit and push to your repository.
3. Submit the repository URL on MCV. If the repository is private, add your instructor as a collaborator.

## What's in this repository

```
AGENTS.md                   rules for every AI agent (+ GEMINI.md, CLAUDE.md, .github/copilot-instructions.md pointers)
prompts/                    the four task files you ask the agent to follow
docs/
  adr/template.md           the ADR format we use
  adr/0001-…                an example ADR (used in Task 3)
  scenario.md               business context for Task 3
exercise/                   your worksheets (you edit these)
  outputs/                  files written by the AI agent
system/online-boutique/     the system you analyse (trimmed snapshot)
```

## Credits

The system under study is [Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo) by Google, used under the Apache License 2.0. See [`system/README.md`](system/README.md) for what was changed for this exercise.

The exercise design draws on Bucaioni et al., *AI Support for Software Architecture Practice: A Systematic Review and Future Directions*, ACM TOSEM 35(9), 2026.
