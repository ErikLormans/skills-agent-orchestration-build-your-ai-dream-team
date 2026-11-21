# Project Pulse – Final Handoff

Project Pulse is a static dashboard for Mona's team. It shows each project as a card with name, owner, status badge, recent activity, priority and an optional summary. A custom agent team built it, coordinated by the Orchestrator.

## Agent team and roles

| Agent | Role in this build |
|-------|--------------------|
| **Orchestrator** | Coordinated the phases (contract → build → integration review → fixes → validation). Ran the Designer and Coder in parallel in Phase 1 because their file scopes did not overlap. Implemented nothing itself. |
| **Planner** | Wrote `docs/project-pulse-plan.md`: file assignments, the shared class and data contracts, dependencies, parallel or sequential decisions, edge cases, validation expectations and open questions. |
| **Designer** | Owned `app/styles.css`. Did the integration review, which led to these fixes: a global `[hidden]` rule, a grid `min()` fix so narrow screens don't overflow, removal of the misleading hover lift (cards aren't clickable), and a `color-mix` fallback border on status badges. |
| **Coder** | Built `app/index.html`, `app/project-data.json` and `.vscode/launch.json`, then ran validation. |

## Delivered files

- `app/index.html`: page markup, `<title>Project Pulse</title>`, and an inline script. The script fetches `project-data.json`, validates it, and renders one `.project-card` per project using `textContent` only (no `innerHTML`). It also handles loading, empty and error states.
- `app/styles.css`: responsive card grid, status badges and priority labels (text plus a non-color cue), focus styles, `prefers-reduced-motion` and `forced-colors` support.
- `app/project-data.json`: 5 projects covering every status (On Track, At Risk, Blocked, Done) and every priority (High, Medium, Low), including one long name to test wrapping.
- `.vscode/launch.json`: one configuration, "Run Project Pulse Dashboard". It runs `py -m http.server 5500` in `app/`. The learner chose `py` on purpose so it runs on Windows, which is a deliberate change from the plan's `python3`.

## Commits

| Commit | Content |
|--------|---------|
| `c267bec` | Agent team doc (`docs/agent-team.md`) |
| `852c5d2` | Plan (`docs/project-pulse-plan.md`) |
| `5c8bcd4` | Dashboard (`app/*`, `.vscode/launch.json`) |

The `py -m http.server 5500` command is already in commit `5c8bcd4`. `git diff` shows no uncommitted changes to `.vscode/launch.json`.

## Dashboard validation

Run on Windows (PowerShell, Python, Node v26.12.1) on 2026-11-21.

| # | Check | Result | Details |
|---|-------|--------|---------|
| 1 | Strict JSON: `python -m json.tool` on `app/project-data.json` and `.vscode/launch.json`; no comments in launch.json | ✅ Pass | Both files exit 0. No `//` or `/* */` in launch.json. |
| 2 | Data shape: top-level `projects` array, and every entry has `name`, `owner`, `status`, `recentActivity`, `priority` | ✅ Pass | 5 projects, no missing fields. |
| 3 | `app/index.html` keyphrases: `<title>Project Pulse</title>`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority`; uses `textContent` | ✅ Pass | All present. `innerHTML` does not appear anywhere. |
| 4 | `app/styles.css` keyphrases: `.dashboard`, `.project-card`, `border-radius`, `box-shadow`, `@media`, `[hidden]` | ✅ Pass | All present. |
| 5 | `.vscode/launch.json`: name "Run Project Pulse Dashboard", cwd `${workspaceFolder}/app`, command, uriFormat `http://localhost:%s/index.html` | ✅ Pass | Name, cwd and uriFormat match. The command is `py -m http.server 5500`. The learner chose this on purpose for Windows, instead of the plan's `python3`. |
| 6 | Serve `app/` on port 5503, then GET `/index.html`, `/styles.css` and `/project-data.json` | ✅ Pass | All 3 returned 200. Server stopped by PID and the port no longer answers. |
| 7 | `node --check` on the extracted inline script | ✅ Pass | Exit 0, no syntax errors. |
| — | Rendering in a real browser (layout, contrast, keyboard) | ⚪ Not run | Needs a manual check. See Known risks / open items. |

## Final handoff

### How to run

1. Open the repo in VS Code on Windows (the `py` launcher must be installed).
2. Open **Run and Debug** (Ctrl+Shift+D).
3. Select **"Run Project Pulse Dashboard"** and press ▶ (F5).
4. The configuration runs `py -m http.server 5500` in `app/`, and the browser opens **http://localhost:5500/index.html** with the dashboard.

### What's delivered

A working static dashboard with 5 sample projects. It has status badges and priority labels, a responsive layout, accessible states, and a one-click launch configuration. The files are listed under Delivered files above.

### Next steps

1. Do the manual browser check at 320, 768 and 1280px widths. Also check keyboard focus and badge contrast (DevTools or Lighthouse).
2. Decide on the open questions from the plan (below).
3. Optional: sort cards by priority, or add a second launch configuration that uses `python3` for Linux or macOS.

## Known risks / open items

- **`py` launcher is Windows-only:** the command `py -m http.server 5500` only works on Windows. In a Linux Codespace or on macOS, the command would need to be `python3 -m http.server 5500`.
- **Exercise checks may expect `python3`:** the plan (validation item 5) expects the literal `python3 -m http.server 5500`. If the exercise's automated checks look for that string, they could flag the `py` command, even though the change was deliberate.
- **Port 5500 conflicts:** if another process (for example Live Server) is using 5500, the server fails to start and the browser never opens. Stop the conflicting process. The plan uses a fixed port with no fallback.
- **Opening via `file://`:** `fetch` is blocked, so the page shows the "Couldn't load project-data.json…" error message. This is by design. Always use the launch configuration.
- **Not checked in a real browser:** validation was static analysis plus HTTP 200 checks. Layout, contrast and keyboard behavior still need the manual check at 320/768/1280px.
- **Open questions from the plan:**
  1. Keep the optional `summary` field? (Currently included and shown only when present.)
  2. Keep port 5500 fixed, or add a fallback?
  3. Add a separate `python3` launch configuration for Linux or macOS next to the `py` one?
  4. Keep the `node-terminal` launch type, or switch to a task plus `preLaunchTask`?
  5. Keep the `Done` status, or show only active projects?
  6. Sort cards (for example by priority), or keep the data file order?

## Git control

None of the agents (Orchestrator, Planner, Designer, Coder) stage, commit or push changes. The learner controls all git operations through Copilot CLI prompts. This file has not been staged or committed.
