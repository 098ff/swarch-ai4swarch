**Language:** [ภาษาไทย](00-prompt-log.md) | English

# Prompt log

**Name / Student ID:**
**Agent(s) used:** e.g. GitHub Copilot (Agent mode), Gemini Code Assist, Gemini CLI, Claude Code, Codex, Cursor, Cline: …
**Model(s):** …
**Plan / tier:** e.g. Copilot Student, Copilot Free, Gemini free tier: …

Log every request you send to the agent. "Files the agent read" is what the agent reported, or what you saw it open.

| # | Task / run | Agent + model | Message sent (short) | Files the agent read | Stayed within the allowed evidence? | Useful? (1–5) | Notes |
|---|---|---|---|---|---|---|---|
| 1 | 1-A | GitHub Copilot (Agent mode, Auto) | `Follow prompts/1-…` run: A, evidence: `demo.proto` | `AGENTS.md`, `1-recover-architecture.md`, `demo.proto`, `1A-architecture.md` | Yes (read only evidence and prompt files, no source code) | 5 | Extracted 9 service contracts accurately; avoided hallucinating runtime dependencies from proto interface |
| 2 | 1-B | GitHub Copilot (Agent mode, Auto: gpt-6-luna) | `Follow prompts/1-…` run: B, evidence: `demo.proto`, `kubernetes-manifests/` | `AGENTS.md`, `1-recover-architecture.md`, `demo.proto`, `kubernetes-manifests/*.yaml` | Yes (read only demo.proto and k8s manifests as allowed) | 4 | Recovered 12 containers and complete runtime dependencies, but hung at mermaid validation |
| 3 | 2-a | | `prompts/2-…` run a | | | | |
| 4 | 2-b | | `prompts/2-…` run b | | | | |
| 5 | 3 | | `prompts/3-…` | | | | |
| 6 | | | | | | | |
| 7 | | | | | | | |
