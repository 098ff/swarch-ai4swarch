**Language:** [ภาษาไทย](README.md) | English

# AI for Software Architecture: Hands-on Exercise

**2110521 Software Architecture · Week 12 · Tutorial 5 / Assignment 5**

You'll use **GitHub Copilot in VS Code, in Agent mode,** as an architecture assistant on a real microservices system, [Online Boutique](system/online-boutique/). Then you'll judge how good its work is. The goal is not to get the AI to do the assignment. The goal is to find out **where AI helps an architect and where it can't be trusted**, which is the main finding of this week's lecture.

You will:

1. **Recover** the architecture from the code (lecture topic T7: architecture recovery)
2. **Recover a decision** that was never written down (T5: decision recovery, knowledge vaporization)
3. **Make a new decision** with an AI-written ADR, with and without context (T3: ADR generation)
4. **Review** the ADR the way an architecture board would, then take responsibility for the final version (AICH4, "Chief Strategist")

Expected time: about 75 minutes in class.

---

## Before class: set up (≈ 15 minutes, do it early)

1. **Get Copilot.**
   - Verified students get Copilot free through [GitHub Education](https://docs.github.com/en/copilot/how-tos/copilot-on-github/set-up-copilot/enable-copilot/set-up-for-students). Approval and activation **can take several days**, so apply now.
   - Copilot Student is **recommended**: Agent mode uses more credits than plain chat. **Copilot Free** works for the core tasks if you're careful (see [Copilot budget](#copilot-budget)).
2. **Install [VS Code](https://code.visualstudio.com/)** and sign in to GitHub inside VS Code (Accounts menu, bottom left).
3. **Create your own copy of this repository.**
   - Click **Use this template → Create a new repository**. It can be private.
   - Clone it and open the folder in VS Code.
4. **Install the recommended extensions** when VS Code asks (listed in [`.vscode/extensions.json`](.vscode/extensions.json)):
   - **GitHub Copilot Chat**
   - **Markdown Preview Mermaid Support**, which renders the diagrams you will produce
5. **Check that it works.**
   1. Open the Chat view (`Ctrl+Alt+I` / `⌃⌘I`).
   2. Pick **Agent** in the agent picker (the dropdown below the chat input).
   3. Type `/` and confirm you can see the four prompts: `/recover-architecture`, `/recover-decision`, `/write-adr`, `/review-adr`.

> Everyone uses the same prompts. They live in [`.github/prompts/`](.github/prompts/), and the shared assistant rules are in [`.github/copilot-instructions.md`](.github/copilot-instructions.md). Read both: understanding what the AI was told is part of judging what it says.

## Copilot budget

The core tasks need about **8–10 agent requests**. Optional Run C in Task 1 adds one more, but it's an expensive one.

- Each agent request reads files and writes a result, so it costs more than a chat answer. Copilot bills by tokens, so **reading less is cheaper**.
- **Start a new chat for every run.** Old conversations are sent again with each request and cost credits.
- Leave the model on **Auto**, unless your instructor says otherwise.
- **You control the evidence.** Each prompt asks which files the agent may read. Which evidence you allow is one of the things this exercise is about.

## How Agent mode works here

- You run a prompt with `/` and fill in its inputs. The agent then reads the allowed files and **writes its result to `exercise/outputs/`**.
- Review the new file, then click **Keep**, or **Undo** if it changed something it shouldn't.
- The agent must not edit `system/`, `docs/adr/0001-*` or your worksheets, and must not run terminal commands. These rules are in [`.github/copilot-instructions.md`](.github/copilot-instructions.md). If the agent asks to run a command, decline.

## Rules of the game

- **Log every request** in [`exercise/00-prompt-log.md`](exercise/00-prompt-log.md): what you asked, which files the agent read, and whether it stayed within the evidence you allowed.
- **Check claims against the code.** A claim without a file path as evidence is a guess until you verify it.
- **Never paste AI output as your own conclusion** without marking it. The worksheets have separate places for *the AI's answer* and *your judgement*.

## The tasks

| # | Worksheet | Prompt | Requests |
|---|---|---|---|
| 1 | [Recover the architecture](exercise/01-recover-architecture.md) | `/recover-architecture` | 2 (+1 optional) |
| 2 | [Recover an unwritten decision](exercise/02-recover-decision.md) | `/recover-decision` | 1 |
| 3 | [Write an ADR for a new requirement](exercise/03-write-adr.md) | `/write-adr` | 2 |
| 4 | [Review it and take ownership](exercise/04-review-adr.md) | `/review-adr` | 1 |

> The worksheets are in English. You may answer in Thai or English.

## Submission (Assignment 5, due by the end of class)

1. Fill in all worksheets in `exercise/`. Keep the agent's files in `exercise/outputs/`, and write your final ADR at `docs/adr/0002-*.md`.
2. Commit and push to your repository.
3. Submit the repository URL on MCV. If the repository is private, add your instructor as a collaborator.

## What's in this repository

```
.github/
  copilot-instructions.md   shared rules for Copilot in this repo
  prompts/                  the four slash-command prompts
docs/
  adr/template.md           the ADR format we use
  adr/0001-…                an example ADR (used in Task 3)
  scenario.md               business context for Task 3
exercise/                   your worksheets (you edit these)
  outputs/                  files written by the Copilot agent
system/online-boutique/     the system you analyse (trimmed snapshot)
```

## Credits

The system under study is [Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo) by Google, used under the Apache License 2.0. See [`system/README.md`](system/README.md) for what was changed for this exercise.

The exercise design draws on Bucaioni et al., *AI Support for Software Architecture Practice: A Systematic Review and Future Directions*, ACM TOSEM 35(9), 2026.
