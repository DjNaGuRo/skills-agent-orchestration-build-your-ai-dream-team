# Project Pulse final handoff

Mona's Project Pulse dashboard is a static frontend built with a custom agent team, orchestrated from GitHub Copilot CLI in a Codespace. Plain HTML, CSS and vanilla JS; no build step.

## Agent team

| Agent | Role in this build |
|-------|--------------------|
| Orchestrator | Defined the shared class-name and data-shape contract, delegated with explicit file scopes, ran Designer and Coder in parallel, and checked the integrated result. |
| Planner | Researched the repo and produced `docs/project-pulse-plan.md`. |
| Designer | Wrote the visual design and accessibility treatment in `app/styles.css`. |
| Coder | Wrote `app/index.html`, `app/project-data.json` and `.vscode/launch.json`. |

Agent definitions live in `.github/agents/`. Team summary: `docs/agent-team.md`. Plan: `docs/project-pulse-plan.md`.

## Deliverables

| File | Owner | Notes |
|------|-------|-------|
| `app/index.html` | Coder | Title "Project Pulse". Links `styles.css`, fetches `project-data.json`, renders one `article.project-card` per project showing status, recentActivity and priority. Uses `textContent` only; handles fetch, JSON, missing-array and empty-list errors. |
| `app/styles.css` | Designer | `.dashboard` responsive grid (`auto-fill`/`minmax`), `.project-card` with `border-radius` and `box-shadow`, status badges and priority tags that do not rely on colour alone, `:focus-visible`, `prefers-reduced-motion`, fallbacks for unknown values. |
| `app/project-data.json` | Coder | Top-level `projects` array with 6 records. Each has name, owner, status, recentActivity and priority, plus an optional summary. |
| `.vscode/launch.json` | Coder | Strict JSON. Launch name "Run Project Pulse Dashboard". |

## How to run

Run the launch configuration "Run Project Pulse Dashboard" from `.vscode/launch.json`. It runs `python3 -m http.server 5500` with `cwd` set to `${workspaceFolder}/app`. `serverReadyAction` opens `http://localhost:%s/index.html`, so the dashboard opens rather than a directory listing. Opening `app/index.html` via `file://` will show an error message, because `fetch()` needs HTTP.

## Final validation

Checked from the command line:

- `app/project-data.json` and `.vscode/launch.json` parse as strict JSON. The launch file has no comments; its only `//` is the `http://` in the URL.
- All 6 projects have the five required fields. Statuses and priorities are varied and within the agreed vocabulary (status: active, at-risk, blocked, paused, completed; priority: high, medium, low).
- With `app/` served on port 5500, `/index.html`, `/styles.css` and `/project-data.json` return HTTP 200, and the page contains `<title>Project Pulse</title>`.
- `index.html` references `styles.css` and `project-data.json`; `styles.css` contains `.dashboard` and `.project-card`.
- The contract class names used in `index.html` all have matching selectors in `styles.css`.
- The test server was stopped afterwards.

Not yet verified:

- Visual rendering in a browser and responsive behaviour at 320px, 768px and 1280px.
- WCAG AA contrast of the badges (chosen for AA but not measured).
- Keyboard focus rings and 200% zoom.
- Running the launch configuration from VS Code.
- Port 5500 may be in use in some environments.

## Final handoff notes

- The dashboard files were committed and pushed to `main` before this review; this handoff file is new and not yet committed.
- Agents did not control git; staging, commits and pushes were done on the learner's instruction.
- Open items from the plan that were resolved by default: summary is an optional field, the script is inline in `app/index.html`, and there is no filter or header counts.
- Suggested next steps: open the dashboard via the launch configuration, do the browser and contrast checks above, and adjust the data in `app/project-data.json`.
