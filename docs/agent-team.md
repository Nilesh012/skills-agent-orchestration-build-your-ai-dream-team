# Agent team

I am using GitHub Copilot CLI in a Codespace to orchestrate a team of custom agents that build Mona's Project Pulse dashboard.

| Agent | Target model | Responsibility | Definition |
|---|---|---|---|
| Orchestrator | Claude Opus 4.7 (copilot) | Breaks down the request, delegates to specialists with explicit file scopes, runs work in parallel only when scopes don't overlap, and verifies the integrated result. Does not implement. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 4.7 (copilot) | Researches the repo and docs, then returns an ordered plan with file assignments, dependencies, edge cases, and validation expectations. Writes no code. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements the logic and runnable-app support (e.g. `.vscode/launch.json` with `cwd` of `${workspaceFolder}/app`) within its assigned files, then validates. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.1 Pro (copilot) | Handles UI/UX, accessibility, and visual design: a polished, responsive dashboard with project cards, status badges, and `.dashboard` / `.project-card` hooks. | `.github/agents/designer.agent.md` |

All agents leave git operations (stage, commit, push) to me, via Copilot CLI prompts.
