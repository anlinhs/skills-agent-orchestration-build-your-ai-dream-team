# Agent team

To build Mona's Project Pulse dashboard, I am using a custom team of four agents defined under `.github/agents/`, orchestrated with the GitHub Copilot CLI in a Codespace.

## Orchestrator

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Coordinates the Planner, Coder, and Designer agents. Breaks the request into phases, assigns non-overlapping file scopes, decides what can run in parallel vs. sequentially, and reports the integrated result. Does not write code or plans itself.
- **Definition:** `.github/agents/orchestrator.agent.md`

## Planner

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Researches the codebase, docs, dependencies, and edge cases, then produces an implementation plan (ordered steps, file assignments, dependencies, parallelizable work, risks, and validation expectations) for the Orchestrator to execute. Does not write code.
- **Definition:** `.github/agents/planner.agent.md`

## Coder

- **Model:** GPT-5.5 (copilot)
- **Responsibility:** Implements application logic and structure within the file scope assigned by the Orchestrator, including Project Pulse support files like `.vscode/launch.json` when assigned. Focuses on deterministic, testable, explicit code following existing repo patterns.
- **Definition:** `.github/agents/coder.agent.md`

## Designer

- **Model:** Gemini 3.1 Pro (copilot)
- **Responsibility:** Owns UI/UX, accessibility, information architecture, and visual design for Project Pulse — producing a polished dashboard with project cards, status badges, and deterministic CSS hooks (`.dashboard`, `.project-card`) within its assigned file scope.
- **Definition:** `.github/agents/designer.agent.md`

All four agents are prohibited from staging, committing, or pushing changes; git operations remain under my control via Copilot CLI prompts.
