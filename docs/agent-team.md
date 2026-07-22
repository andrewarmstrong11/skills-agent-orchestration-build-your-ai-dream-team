# Agent team

This is the custom agent team used to build Mona's Project Pulse dashboard, defined under `.github/agents/`.

- **Orchestrator** — Model: Claude Opus 4.7 (copilot). Coordinates the Planner, Coder, and Designer agents: breaks requests into phases, assigns file scopes, runs non-overlapping work in parallel, and reports the final outcome. Defined in `.github/agents/orchestrator.agent.md`.
- **Planner** — Model: Claude Opus 4.7 (copilot). Researches the repository, docs, and dependencies to produce an implementation plan with ordered steps, file assignments, dependencies, parallelizable vs. sequential work, edge cases, and open questions. Does not write code. Defined in `.github/agents/planner.agent.md`.
- **Coder** — Model: GPT-5.5 (copilot). Implements code, logic, and support configuration (e.g. `.vscode/launch.json` for Project Pulse) within the file scope assigned by the Orchestrator. Defined in `.github/agents/coder.agent.md`.
- **Designer** — Model: Gemini 3.1 Pro (copilot). Handles UI/UX, accessibility, information architecture, and visual design, producing the polished Project Pulse dashboard styling (cards, status badges, responsive layout). Defined in `.github/agents/designer.agent.md`.

All work is orchestrated using GitHub Copilot CLI running in a Codespace.
