# Agent team

I am using GitHub Copilot CLI in a Codespace to orchestrate the work of building Mona's Project Pulse dashboard with a team of custom agents.

| Agent | Target model | Responsibility | Definition |
|-------|--------------|----------------|------------|
| Orchestrator | Claude Opus 4.7 (copilot) | Breaks down requests, delegates to specialists with explicit file scopes, runs work in parallel only when scopes don't overlap, and verifies the integrated result. Does not implement. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 4.7 (copilot) | Researches the repo and docs, then returns an ordered plan with file assignments, dependencies, edge cases, and validation expectations. Writes no code. | `.github/agents/planner.agent.md` |
| Designer | Gemini 3.1 Pro (copilot) | Handles UI/UX, accessibility, and visual design, including the polished dashboard styling (`.dashboard`, `.project-card`, badges, responsive layout). | `.github/agents/designer.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements code and logic within its assigned files, validates changes, and creates runnable-app support such as `.vscode/launch.json`. | `.github/agents/coder.agent.md` |

All agents leave git operations (stage, commit, push) to me via Copilot CLI prompts.
