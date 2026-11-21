# Project Pulse Implementation Plan

## Summary

Project Pulse is a static dashboard for Mona's team. It shows each project as a card with its name, owner, status badge, recent activity, priority and an optional short summary. Card data comes from `app/project-data.json` and is drawn by a small script in `app/index.html`. The Designer styles the page in `app/styles.css`. A VS Code launch configuration named **Run Project Pulse Dashboard** serves the `app/` folder with `python3 -m http.server 5500` and opens `http://localhost:%s/index.html`, so learners see the dashboard instead of a directory listing.

The Orchestrator coordinates the work. The Planner wrote this plan. The Designer owns the visual design and accessibility, and the Coder owns markup, script, data and launch configuration. Each file has exactly one owner, and the agents share a class-name contract (below) so the Designer and Coder can work at the same time without editing the same file.

These sources shaped the plan: `.github/project-pulse-brief.md`, `.github/steps/2-step.md` through `4-step.md`, `.github/workflows/2-step.yml` and `3-step.yml` (the keyphrase and JSON checks), `docs/agent-team.md`, and `.github/agents/*.agent.md`.

---

## File assignments

| File | Owner | Purpose |
|------|-------|---------|
| `app/index.html` | **Coder** (Designer reviews structure and accessibility) | Page markup with the exact title "Project Pulse". Links `styles.css` and loads `project-data.json` with `fetch`. Builds one `.project-card` per project inside a `.dashboard` container, showing `name`, `owner`, `status`, `recentActivity` and `priority`. Handles the loading, empty and error states. |
| `app/styles.css` | **Designer** | Polished, responsive look: `.dashboard` and `.project-card` selectors, `border-radius`, `box-shadow`, status badge colors, priority styling, typography, spacing and focus styles. |
| `app/project-data.json` | **Coder** | Strict JSON with a top-level `"projects"` array. Each project has `name`, `owner`, `status`, `recentActivity`, `priority` and optionally `summary`. |
| `.vscode/launch.json` | **Coder** | Strict JSON with no comments. One configuration named **Run Project Pulse Dashboard**. It uses `cwd` `${workspaceFolder}/app`, runs `python3 -m http.server 5500`, and has a `serverReadyAction` that opens `http://localhost:%s/index.html`. |

### Shared contract between `index.html` and `styles.css`

The Orchestrator gives this contract to both agents before Phase 1. Neither agent may rename these hooks without the Orchestrator's approval.

| Hook | Element / meaning |
|------|-------------------|
| `.dashboard` | `<main>` wrapper for the whole dashboard |
| `.dashboard-header` | `<header>` holding the `<h1>Project Pulse</h1>` title and subtitle |
| `.project-grid` | `<ul>` or `<section>` that holds the cards (`role="list"` if it is a `<ul>`) |
| `.project-card` | `<article>` for one project |
| `.project-card__name` | `<h2>` with the project name |
| `.project-card__owner`, `.project-card__activity`, `.project-card__summary` | Text rows on a card |
| `.status-badge` + `.status-badge--on-track` / `--at-risk` / `--blocked` / `--done` | Status pill |
| `.priority` + `.priority--high` / `--medium` / `--low` | Priority label (the text is always visible; color is extra) |
| `.dashboard-message` + `.dashboard-message--error` | Loading, empty and error messages (`role="status"` or `role="alert"`) |

### Data contract (`project-data.json`)

- `status` is one of `"On Track"`, `"At Risk"`, `"Blocked"`, `"Done"`. The script turns it into a modifier class by lowercasing it and replacing spaces with hyphens. `"Done"` becomes `--done`.
- `priority` is one of `"High"`, `"Medium"`, `"Low"`.
- `recentActivity` is a short sentence. A date prefix is optional.
- Include 4–6 projects that cover every status and priority value. Include at least one long name so wrapping gets tested.

---

## Designer responsibilities

- Write `app/styles.css` only.
- Make it look like a finished dashboard, not a plain page:
  - Header area.
  - Card grid using `grid-template-columns: repeat(auto-fill, minmax(280px, 1fr))`.
  - Rounded cards (`border-radius`) with a `box-shadow` and a hover or focus lift.
  - Readable spacing and a clear type scale.
- Status badges: a different color for each status. Text contrast must be at least 4.5:1 (WCAG AA).
- Priority: the label text always shows. Add a visual weight cue for High, such as an accent border or icon-free emphasis. Never use color alone.
- Responsive layout: one column at 480px or below, and comfortable margins on wide screens.
- Long names wrap (`overflow-wrap: anywhere`).
- Visible `:focus-visible` styles. Support `prefers-reduced-motion`.
- Style the `.dashboard-message` states.
- Use CSS custom properties for colors so contrast is easy to tune.
- Advisory only: review the semantic structure in `index.html` (landmarks, heading order, badge text) and send recommendations to the Coder through the Orchestrator. **Do not edit `index.html`.**

## Coder responsibilities

- Write `app/project-data.json` according to the data contract.
- Write `app/index.html`:
  - `<!doctype html>`, `lang="en"`, viewport meta, `<title>Project Pulse</title>`, `<link rel="stylesheet" href="styles.css">`.
  - `<main class="dashboard">` containing the `<h1>Project Pulse</h1>` header.
  - An inline `<script>` that calls `fetch('project-data.json')`, checks `response.ok`, validates the data and builds a `.project-card` for each project using the contract classes.
  - Set text with `textContent` (no `innerHTML` with data) to prevent injection.
  - Read the field names `status`, `recentActivity` and `priority` explicitly in the script. The Step 3 workflow checks for these keyphrases.
  - Show loading, empty and error states in `.dashboard-message`.
- Write `.vscode/launch.json` as strict JSON with no comments:
  - `"version": "0.2.0"`.
  - One configuration with `"name": "Run Project Pulse Dashboard"`, `"type": "node-terminal"`, `"request": "launch"`, `"command": "python3 -m http.server 5500"` and `"cwd": "${workspaceFolder}/app"`.
  - `"serverReadyAction"` with:
    - `"pattern": "Serving HTTP on .* port ([0-9]+)"`
    - `"uriFormat": "http://localhost:%s/index.html"`
    - `"action": "openExternally"`
- Apply the Designer's markup recommendations in Phase 2.
- Run JSON validation and report the results.

---

## Ordered implementation steps

1. **Phase 0 – Contract (Orchestrator):** Send the class-name contract and data contract above to both agents.
2. **Phase 1A – Designer:** Write `app/styles.css` against the contract. Use the sample markup structure in this plan for reference.
3. **Phase 1B – Coder (in order inside this phase):**
   1. Write `app/project-data.json`.
   2. Write `app/index.html` with the render script that consumes that schema.
   3. Write `.vscode/launch.json`.
4. **Phase 2 – Integration review (Designer, read-only):** Check the rendered `index.html` and `styles.css` together for hierarchy, contrast, responsiveness and semantics. Return a list of findings, sorted by file owner.
5. **Phase 3 – Fixes (run in parallel only if both files need changes):** The Designer fixes `styles.css`. The Coder fixes `index.html` or the data file.
6. **Phase 4 – Validation (Orchestrator):** Run the checks in the Validation expectations section. Then report the outcome to the learner.

## Dependencies

- `project-data.json` schema → `index.html` render script. The field names and enum values must be fixed first. The contract handles this, so the Coder writes the data before the HTML.
- Class-name contract → both `index.html` and `styles.css`. This is the only coupling between Designer and Coder.
- `index.html` filename and `app/` location → `launch.json`. The launch config needs `cwd` `${workspaceFolder}/app` and the `/index.html` URL.
- Running over HTTP (launch config) → the `fetch` call working. Opening the file through `file://` breaks `fetch`.
- Phases 1A and 1B → Phase 2 review → Phase 3 fixes → Phase 4 validation.

## Parallel work decisions

- **Parallel:** Phase 1A (Designer, `styles.css`) and Phase 1B (Coder, `project-data.json`, `index.html`, `launch.json`).
  - Why: the file scopes do not overlap, and the shared contract is fixed in Phase 0, so there is no runtime data dependency between them.
- **Sequential inside Phase 1B:** `project-data.json`, then `index.html`, then `launch.json`.
  - Why: they have one owner, and the render script depends on the data schema.
- **Sequential:** Phase 2 runs after both parts of Phase 1. It needs both files to exist.
- **Parallel in Phase 3:** only if the findings touch both `styles.css` and `index.html`. If a fix changes the contract (for example, a new class), the Coder adds the markup first and the Designer styles it afterwards.
- **Sequential:** Phase 4 runs last.

## Edge cases

- **Empty `projects` array:** Show the message "No projects to show yet" (`role="status"`). Do not leave a blank grid.
- **Missing or invalid `projects` key:** Show an error message and log details with `console.error`.
- **Fetch failure** (opened through `file://`, 404, network error, or invalid JSON): Show an error in `.dashboard-message--error` with `role="alert"`, for example: "Couldn't load project-data.json. Start the 'Run Project Pulse Dashboard' launch configuration." Check `response.ok` before parsing.
- **Missing fields:** Show "Unknown" or "—" for that field. An unknown `status` or `priority` gets a neutral badge (`status-badge--unknown`) instead of breaking the page.
- **Long names or long activity text:** Wrap them (`overflow-wrap: anywhere`). Never let them push the card off screen. Truncating with a `title` attribute is optional.
- **Small screens:** One column at 480px or below. Badges wrap. No horizontal scroll at 320px.
- **Accessibility:**
  - Landmarks: `header`, `main`.
  - One `h1`, then an `h2` per card.
  - Status and priority shown as text, not color only.
  - Contrast at least 4.5:1.
  - Visible focus.
  - `prefers-reduced-motion` respected.
  - Loading and error messages announced through `aria-live` or a matching role.
- **Port 5500 already in use:** The server fails to start. The learner should stop the other process. See open question 2.
- **Untrusted data:** Use `textContent` only, never `innerHTML` with data values.

## Validation expectations

1. **JSON validity:**
   - Run `python3 -m json.tool app/project-data.json` and `python3 -m json.tool .vscode/launch.json`. On Windows, use `python` or `py -m json.tool`.
   - Both must parse. `launch.json` must contain no comments and no trailing commas.
2. **Data shape:** There is a top-level `projects` array. Every entry has `name`, `owner`, `status`, `recentActivity` and `priority`.
3. **`index.html` keyphrases:** It contains `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity` and `priority`.
4. **`styles.css` keyphrases:** It contains `.dashboard`, `.project-card`, `border-radius` and `box-shadow`.
5. **`launch.json` content:** It contains `Run Project Pulse Dashboard`, `${workspaceFolder}/app`, `python3 -m http.server 5500` and `http://localhost:%s/index.html`.
6. **Page loads at the launch URL:** Run **Run Project Pulse Dashboard** from Run and Debug. The browser opens `http://localhost:5500/index.html` and shows the dashboard, not a directory listing.
7. **Rendering:** The number of cards matches the number of entries in `projects`. Each card shows all five fields, and the status badge and priority classes are applied.
8. **Error path:** Opening `index.html` through `file://`, or temporarily renaming the data file, shows the error message instead of a blank page.
9. **Responsive check:** At about 320px, 768px and 1280px widths the layout goes from one column to several, with no horizontal scroll.
10. **Accessibility check:**
    - Heading order is correct and landmarks are present.
    - Badge contrast is at least 4.5:1 (check with DevTools or Lighthouse).
    - Keyboard focus is visible.
    - Status and priority can be read without color.
11. **Do not use `scripts/validate-exercise.sh` as a pass/fail gate.** It checks the template itself and reports a failure when learner files such as `app/*` are committed.

## Open questions

1. Should the optional `summary` field ("contributor-friendly summary" in the brief) be included? This plan says yes. It is optional and shown only when present.
2. Is the port fixed at 5500? The Step 3 prompt says so. If 5500 is busy, should there be a fallback, or should the learner just stop the conflicting process? This plan assumes a fixed port.
3. Running locally on Windows: `python3` may not be on PATH (it may be `python` or `py`). The exercise targets a Linux Codespace, so the plan keeps `python3`. Should Windows users get a second launch configuration? Not planned, so the launch file stays deterministic.
4. Should the launch type be `node-terminal`, or a shell task plus a `preLaunchTask`? This plan uses `node-terminal` because it supports `serverReadyAction` directly.
5. Is a `"Done"` status needed, or only active projects? The brief says "which projects are active". This plan includes `Done` so the badge gets tested.
6. Should cards be sorted, for example by priority? This plan keeps the order from the data file. Sorting is an optional improvement.

---

> **Git control:** None of the agents (Orchestrator, Planner, Designer, Coder) stage, commit or push changes. The learner does all git operations through Copilot CLI prompts.
