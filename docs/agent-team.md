# Agent Team for Mona's Project Pulse Dashboard

The custom agents live in `.github/agents/`. The Orchestrator coordinates three specialists. None of them stage, commit, or push; you control git through Copilot CLI prompts.

| Agent | Role | Model | Tools | Writes files? |
|-------|------|-------|-------|---------------|
| **Orchestrator** | Breaks down the request, delegates to specialists, verifies the integrated result. Does not implement. | Claude Opus 5.5 | read, agent, memory | No |
| **Planner** | Researches the repo and docs, then returns steps, file assignments, dependencies, parallel/sequential work, edge cases, validation expectations, and open questions. Writes no code. | Claude Opus 5.5 | read, search, web, memory, todo | No |
| **Coder** | Implements logic and fixes bugs within assigned files. Creates `.vscode/launch.json` when assigned, with `cwd` set to `${workspaceFolder}/app` and `index.html` opened. | GPT-5.5 | read, edit, search, execute, web, memory, todo | Yes |
| **Designer** | Handles UI/UX, accessibility, and visual design. For Project Pulse it builds a polished dashboard with `.dashboard` and `.project-card` hooks, status badges, clear priority, and a responsive layout. | Gemini 3.7 Flash | read, edit, search, web, memory, todo | Yes |

## Workflow

1. The Orchestrator asks the Planner for a plan.
2. It splits the plan into phases using the file assignments and dependencies.
3. It runs tasks in parallel only when file scopes do not overlap and there are no data dependencies. Otherwise it runs them sequentially.
4. Each specialist works only within its assigned file scope. The Coder does not change design-only files, and the Designer stays within its assigned files.
5. The Orchestrator checks that the result fits together and reports the outcome.
