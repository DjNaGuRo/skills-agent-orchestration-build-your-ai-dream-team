# Project Pulse Dashboard: Implementation Plan

## Summary

Build Mona's Project Pulse dashboard as a small static frontend. It shows each project's name, owner, status, recent activity, priority/risk, and a short summary, laid out as polished cards. The `app/` directory is empty, so this is a greenfield build.

**Research findings**
- `.github/project-pulse-brief.md` is the requirements source. It requires `app/index.html`, `app/styles.css`, `app/project-data.json` and `.vscode/launch.json`.
- The data file must have a top-level `projects` array. Each project needs `name`, `owner`, `status`, `recentActivity` and `priority`.
- The brief also asks for a "short contributor-friendly summary". That field is not in the required list, so it is treated as an optional extra field (see Open questions).
- The launch configuration must be named **Run Project Pulse Dashboard**. It must serve from `app/` and open `index.html`, not a directory listing.
- `docs/agent-team.md` and `.github/agents/*.agent.md` define the roles:
  - The Designer owns UI/UX and styling, including the hooks `.dashboard`, `.project-card`, badges and responsive layout.
  - The Coder owns logic and `.vscode/launch.json`. That file must be strict JSON with no comments, with `cwd` set to `${workspaceFolder}/app`, and it must open `index.html`.
  - Agents stay inside their assigned file scope.
  - No agent touches git.
- `.vscode/` currently contains only `tasks.json` (the terminal bootstrap task). `launch.json` does not exist yet and must not disturb `tasks.json`.

**Approach:** plain HTML, CSS and vanilla JS with no build step or dependencies. `index.html` fetches `project-data.json` and renders cards. Because `fetch()` is blocked on `file://`, the launch config must serve `app/` over HTTP.

## Ordered implementation steps

1. **Define the contract (Orchestrator, before delegation).** Fix the data shape and class-name contract (see Dependencies) and include it verbatim in both the Designer's and the Coder's prompts.
2. **Coder: `app/project-data.json`.** Create the sample data using the agreed shape. Include a varied mix of statuses and priorities, plus at least one edge-case record that is valid but long or sparse.
3. **Designer: `app/styles.css`.** Style against the contract's class names. Cover the page shell, header, summary area, card grid, status badges, priority treatment, empty/error states, responsive breakpoints, focus styles and contrast.
4. **Coder: `app/index.html`.** Create the markup shell and an inline `<script>` that fetches `project-data.json`, validates it, and renders cards using exactly the contract's classes and `data-*` attributes. Link `styles.css`.
5. **Coder: `.vscode/launch.json`.** Create strict JSON with a configuration named **Run Project Pulse Dashboard**. Set `cwd` to `${workspaceFolder}/app`, serve `app/` over HTTP, and open `index.html`.
6. **Integration check (Orchestrator).** Run the validation checklist below, assign any fixes to the owning agent only, and re-verify.

## File assignments (table)

| File | Owner | Purpose |
|------|-------|---------|
| `app/index.html` | Coder | Page markup plus the logic that loads and renders the project data into cards. Links `styles.css`. |
| `app/styles.css` | Designer | All visual design: `.dashboard`, `.project-card`, status badges, priority treatment, responsive layout, accessibility styling. |
| `app/project-data.json` | Coder | Sample project data with a top-level `projects` array. Consumed by `index.html`. |
| `.vscode/launch.json` | Coder | Strict-JSON launch config "Run Project Pulse Dashboard". It serves `app/` (`cwd` is `${workspaceFolder}/app`) and opens `index.html`. |

No other files should be created or modified. This includes `.vscode/tasks.json`, `docs/agent-team.md` and the agent definitions. Saving this plan as `docs/project-pulse-plan.md` is a separate action for the learner or Orchestrator.

## Designer responsibilities

- Own `app/styles.css` only. Do not edit HTML, JSON or launch files.
- Give the first view a clear "Project Pulse dashboard" look, not a bare page. Use rounded corners, shadows, strong contrast, clear typography and generous spacing.
- Use this hierarchy on each card: project name, then status badge and priority marker, then owner, then recent activity, then summary.
- Style the required hooks:
  - `.dashboard` is the card grid container, using CSS grid with `auto-fill`/`minmax`.
  - `.project-card` is the card.
  - Status badges use `.status-badge` with modifier classes, one per status value.
  - Priority treatment uses `.priority` with modifier classes, one per priority level.
  - Cover the empty and error states and the summary counts, if the Coder renders them.
- Do not rely on color alone for status or priority. The badge text carries the meaning, and styling may add a border, icon-like marker or weight difference.
- Meet WCAG AA contrast, add visible `:focus-visible` styles, and use readable base font sizes.
- Make the layout responsive: one column on mobile, two or three columns on larger screens, with no horizontal scroll at 320px.
- Include a fallback style for unknown status or priority values.
- Report design decisions, tradeoffs and validation recommendations.

## Coder responsibilities

- Own `app/index.html`, `app/project-data.json` and `.vscode/launch.json` only. Do not edit `styles.css`.
- `project-data.json`:
  - Use a top-level `projects` array of 5–7 records.
  - Required fields are `name`, `owner`, `status`, `recentActivity` and `priority`.
  - Include an optional `summary` string for the contributor-friendly summary.
  - Use a small fixed vocabulary, for example status: `active | at-risk | blocked | paused | completed`, and priority: `high | medium | low`. These values must be confirmed in the contract.
  - The data must be valid JSON with no comments.
- `index.html`:
  - Provide semantic markup: `<header>`, `<main>`, a heading hierarchy, and a `<meta viewport>`.
  - Link `styles.css` with a relative path.
  - Fetch `project-data.json` with a relative path and render cards into `.dashboard`.
  - Set text with `textContent` rather than `innerHTML`, so data cannot inject markup.
  - Normalize status and priority to a lowercase, hyphenated class suffix.
  - Handle fetch failure, invalid JSON, a missing or non-array `projects`, and an empty list. Show an explicit, readable message in each case.
  - Give the loading state a deterministic container.
  - Keep the logic simple, explicit and descriptive.
- `.vscode/launch.json`:
  - Use strict JSON with no comments or trailing commas, and `"version": "0.2.0"`.
  - Name the configuration exactly `Run Project Pulse Dashboard`.
  - Set `cwd` to `${workspaceFolder}/app`.
  - Serve over HTTP on a fixed port, and open `http://localhost:<port>/index.html`. Prefer a fixed port, a deterministic command and `serverReadyAction`, rather than a bare directory-listing URL.
  - Pick a serving approach that needs no install, for example `python3 -m http.server <port>`. Confirm that Python or Node is available in the devcontainer (see Open questions).
- Report what changed, what was validated and any remaining risk.

## Dependencies between steps

**Shared contract (the key dependency).** The Orchestrator should fix this before delegating:

- **Data shape.** `{ "projects": [ { "name", "owner", "status", "recentActivity", "priority", "summary"? } ] }`. All fields are strings.
- **Enumerations.** Status and priority vocabularies as listed above.
- **Classes and hooks.** Both the Coder (generates them) and the Designer (styles them) must use these names:
  - `.dashboard` for the card container.
  - `.project-card` for each card.
  - `.project-name`, `.project-owner`, `.project-activity` and `.project-summary` for card fields.
  - `.status-badge` plus `.status-<value>`, for example `.status-at-risk`.
  - `.priority` plus `.priority-<value>`, for example `.priority-high`.
  - `.state-message` plus `.state-error` and `.state-empty` for the loading, empty and error messages.
- **Page-level hooks.** The page header and summary hooks follow the same naming style and must be listed in the contract, for example `.page-header`.

**Step dependencies**
- Step 1 (contract) blocks steps 2, 3 and 4.
- Step 4 needs the data shape from step 2, or the contract alone. It does not need the finished CSS.
- Step 3 needs only the contract, not the finished HTML.
- Step 5 depends on nothing except the existing `app/` directory and the `index.html` filename. It is verified in step 6 against the real files.
- Step 6 depends on steps 2–5.

## Parallel work decisions

**Can run in parallel (after the contract exists)**
- **Designer on `app/styles.css`** and **Coder on `app/project-data.json`, `app/index.html` and `.vscode/launch.json`**.
  - Justification: the file scopes are disjoint, so there is no write conflict. The only coupling is the class-name and data-shape contract, and both sides code against it rather than against each other's output.
- Within the Coder's scope, the three files are independent given the contract. They are owned by one agent, so the Coder should do them sequentially (JSON, then HTML, then launch).

**Must be sequential**
- **Contract definition before both tracks.** If the two agents invent names independently, the CSS selectors will not match the rendered markup.
- **Integration verification after all files exist.** It depends on all four files and on the contract being honored.
- **Fix rounds.** A fix to a contract mismatch goes to the owner of the file that deviated. If the contract itself must change, re-sync both agents before continuing, and do not run the tracks in parallel while it is changing.

**Risk if the parallel assumption fails:** if the Designer needs a markup change, such as an extra wrapper element, the Orchestrator should handle it as a contract amendment, not as an edit to `index.html` by the Designer.

## Edge cases

- **`file://` fetch is blocked.** Opening `index.html` directly shows an error. The launch config must serve over HTTP, and the UI should show a helpful message that mentions this if the fetch fails.
- **Directory listing instead of the dashboard.** The URL must end in `/index.html`, or the server must serve `index.html` by default.
- **Port conflicts.** A fixed port may already be in use.
- **Data problems:**
  - `projects` is missing, is not an array, or is empty.
  - A project is missing a field or has an empty string or `null` for one. Render a fallback such as "Unassigned" or "No recent activity" instead of `undefined`.
  - An unknown status or priority value. Render it with the fallback badge style and a safe class.
  - Very long names or activity text should wrap, not overflow.
  - Special characters or HTML in strings must not be interpreted as markup.
- **Class-suffix normalization.** Values with spaces or capitals, such as "At Risk", must map to `at-risk`.
- **Accessibility:** do not rely on color alone, keep contrast AA, preserve keyboard focus visibility, keep headings in order, and consider `prefers-reduced-motion` for any transitions.
- **Responsive:** 320px width, very wide screens (cap the container width), and print or zoom at 200%.
- **JSON validity:** a trailing comma or comment in `project-data.json` or `launch.json` breaks parsing. VS Code tolerates comments in `launch.json`, but the requirement is strict JSON.
- **Existing `.vscode/tasks.json`:** do not overwrite or alter it.

## Validation expectations

- `app/project-data.json` and `.vscode/launch.json` parse as strict JSON, for example with `python3 -m json.tool` or `node -e "JSON.parse(...)"`.
- `launch.json` has the configuration named exactly "Run Project Pulse Dashboard", `cwd` equal to `${workspaceFolder}/app`, and a URL or path that opens `index.html`.
- Serving `app/` over HTTP returns 200 for `/index.html`, `/styles.css` and `/project-data.json`.
- The rendered page shows one `.project-card` per project, with name, owner, status badge, priority treatment, recent activity and summary visible. The cards sit in a `.dashboard` container.
- The class names used in `index.html` match the selectors in `styles.css`. A grep for each contract class in both files is a cheap check.
- Check the error path by temporarily pointing at a missing data file, or by opening the page via `file://`. A readable message should appear and the console should have no uncaught exceptions. Restore the file afterwards.
- Responsive check at about 320px, 768px and 1280px, with no horizontal scrolling.
- Accessibility spot check: keyboard tab order and focus ring, contrast of badges, and heading order.
- No files outside the four assigned files changed. The learner can confirm this in `git status`, and no agent stages or commits anything.
- Launching from VS Code is the final manual check, if available. It should open the dashboard in a browser rather than a listing.

## Open questions

1. **Serving mechanism for `launch.json`.** The brief says to serve from `app/` and open `index.html`, but not which runtime to use. This is unresolved, and I could not verify what the devcontainer provides beyond what I read. Options are `python3 -m http.server` via a `node-terminal` configuration, or `npx http-server`. The Coder must pick one that works without installing anything extra, and should check `.devcontainer` first. The `serverReadyAction` options and the exact launch type also need verifying against current VS Code documentation.
2. **Port number.** Any fixed port will do, for example 8000 or 5500. It should be confirmed to be free and forwardable in Codespaces.
3. **Status and priority vocabularies.** The brief does not define them. The values proposed here are assumptions, and the Orchestrator should confirm or change them before the contract is distributed.
4. **`summary` field.** The brief asks for a short summary but does not list it among the required fields. This plan treats it as an optional extra field. Confirm that is acceptable.
5. **Page-level extras.** Decide whether to include a header summary, such as counts by status, or a filter. This plan assumes the header and counts are optional and the filter is out of scope. Anything extra needs a contract entry for both the Designer and the Coder.
6. **Inline vs. external script.** Only `index.html` is allowed, so the logic is assumed to be inline. If the learner wants a separate `app.js`, that adds a fifth file and breaks the "exactly these files" constraint.
7. **Docs.** Whether this plan should also be referenced from `docs/agent-team.md` is left to the learner. It is out of scope here.
