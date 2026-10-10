# Project Pulse Dashboard: Implementation Plan

Created by the Planner agent.

## Summary

The repo is a template: `app/` is empty and `.vscode/` holds only `tasks.json`. The build is a static app with no build step and no dependencies, across four files and two owners:

- **Coder** owns `app/project-data.json`, `app/index.html` (structure and data loading) and `.vscode/launch.json`.
- **Designer** owns `app/styles.css` and the design and accessibility guidance.

## Hard requirements

- `app/index.html`: title is exactly "Project Pulse"; references `styles.css` and `project-data.json`; renders visible cards with class `project-card`; each card shows status, recentActivity and priority.
- `app/styles.css`: contains `.dashboard` and `.project-card` selectors; uses `border-radius` and `box-shadow`; responsive layout.
- `app/project-data.json`: top-level `"projects"` array; each project has `name`, `owner`, `status`, `recentActivity` and `priority`.
- `.vscode/launch.json`: strict JSON (no comments, no trailing commas); configuration named "Run Project Pulse Dashboard"; `cwd` is `${workspaceFolder}/app`; command `python3 -m http.server 5500`; `serverReadyAction` opens `http://localhost:%s/index.html`, so the dashboard opens rather than a directory listing.

## Ordered steps and file assignments

| # | Step | Owner | Files (exclusive) | Depends on |
|---|---|---|---|---|
| 1 | **Data contract and sample data.** Create the `projects` array with 5-6 realistic Mona-team projects covering a mix of statuses (Active, At Risk, Blocked, On Hold, Completed) and priorities (High, Medium, Low). Use short, human-readable `recentActivity` strings. Share the enum values with the Designer. | Coder | `app/project-data.json` | none |
| 2a | **Design spec.** Hand over the class and data-attribute contract: hook names, how status and priority map to badge variants, and accessibility requirements. | Designer | none (written handoff) | Step 1 enum values, or an agreed list |
| 2b | **Markup and data loading.** Semantic HTML: header with the `Project Pulse` h1, a `main.dashboard` container, and a status/loading region. Use `fetch('project-data.json')` to render one `article.project-card` per project. Add empty-state and error messages. Label each field. Add status and priority badges using the agreed hooks. Use `textContent`, not `innerHTML`. Link `styles.css`. | Coder | `app/index.html` | Steps 1 and 2a |
| 3 | **Styling.** `.dashboard` as a responsive CSS grid; style `.project-card` with `border-radius`, `box-shadow` and a type scale. Status badge variants and a priority treatment (for example a left accent border plus a text label, never colour alone). `:focus-visible` styles, contrast-safe palette, `prefers-reduced-motion` handling, and loading, error and empty states. | Designer | `app/styles.css` | Step 2a (can run in parallel with 2b) |
| 4 | **Launch config.** Strict JSON with `"version": "0.2.0"` and a configuration named "Run Project Pulse Dashboard", with `cwd` `${workspaceFolder}/app` and command `python3 -m http.server 5500`. Add `serverReadyAction` with a `pattern` matching "Serving HTTP on ... port 5500", `uriFormat` `http://localhost:%s/index.html` and `action` `openExternally`. | Coder | `.vscode/launch.json` | none |
| 5 | **Integration check and review.** Run the validation checks. The Designer reviews the rendered output; the Coder fixes markup, data or launch issues. | Orchestrator, then each owner for their own file | all four files, one owner at a time | Steps 1-4 |

## Responsibilities

**Designer**
- Own `app/styles.css`.
- Provide the design spec: hook names, badge and priority mapping, accessibility requirements.
- Deliver a polished, responsive dashboard: visible cards, status badges, clear priority treatment, rounded corners, shadows, readable spacing.
- Review the rendered result at several viewport sizes.
- Route markup suggestions through the Coder rather than editing `index.html`.

**Coder**
- Own `app/project-data.json`, `app/index.html` and `.vscode/launch.json`.
- Implement semantic markup and the fetch-and-render logic, including empty, error and unknown-value handling.
- Create strict-JSON launch configuration and prove it works by running it.
- Validate changes and report what changed, what was validated and any remaining risk.

## Dependencies

- Step 1 (data) gives Step 2a the status and priority values.
- Step 2a (design spec) gives Steps 2b and 3 the shared class and data-attribute contract.
- Step 2b needs Steps 1 and 2a; Step 3 needs Step 2a only.
- Step 4 (launch config) is independent.
- Step 5 needs Steps 1-4.

## Parallel and sequential decisions

- **Phase A (parallel):** Coder on `app/project-data.json` and `.vscode/launch.json`; Designer on the design spec (no files). Scopes do not overlap.
- **Phase B (parallel):** Coder on `app/index.html`; Designer on `app/styles.css`. Different files; the agreed class names are the contract, so the Orchestrator must pass the same hook list to both.
- **Phase C (sequential):** Orchestrator verifies. Fixes go to a single owner per file, with no concurrent edits.
- **Never parallelise** two tasks that edit the same file.
- **Fallback:** if the contract risk is a concern, run Step 2b first, then Step 3.

## Class and data contract (fixed before Phase B)

- Required hooks: `.dashboard`, `.project-card`.
- Suggested hooks: `.project-card__header`, `.badge`, `.badge--status-<slug>`, `.badge--priority-<slug>`, and `data-status` / `data-priority` attributes on the card.
- Slugs are lower-case, hyphenated versions of the data values, from a fixed shared list.

## Edge cases

- `fetch` fails over `file://`: show a clear error pointing to the launch config.
- Fetch failure, non-OK response, malformed JSON or missing `projects`: visible accessible error (`role="alert"`), never a blank page.
- Empty `projects` array: empty-state message.
- Missing fields: fall back to "Unknown" or "Not set" and a neutral badge.
- Unexpected status or priority values: default badge style.
- Long names or activity text: wrap without overflow (`min-width: 0` in the grid).
- Port 5500 already in use: note in the handoff; do not change the port.
- `serverReadyAction` pattern must match Python's actual output and capture the port; test it.
- Accessibility: one `h1`, a heading per card, badge text as real text, 4.5:1 contrast, visible focus, responsive down to about 320px, `aria-live` on the status region.
- Strict JSON in both `launch.json` and `project-data.json`.
- Agents do not stage, commit or push; the learner controls git.

## Validation expectations

1. `python3 -m json.tool app/project-data.json` and `python3 -m json.tool .vscode/launch.json` both succeed.
2. `projects` is an array and every entry has all five required keys with non-empty values from the agreed lists.
3. `index.html` contains `Project Pulse`, `styles.css`, `project-data.json` and `project-card`; `styles.css` contains `.dashboard`, `.project-card`, `border-radius`, `box-shadow` and a media or container query.
4. `launch.json` contains "Run Project Pulse Dashboard", `${workspaceFolder}/app`, `python3 -m http.server 5500`, `http://localhost:%s/index.html` and `serverReadyAction`.
5. Serve `app/` on port 5500, open `/index.html`, and confirm cards render with all fields and no directory listing appears. Also run the launch configuration in VS Code.
6. Temporarily break the data to check the empty, error and `file://` states, then restore it.
7. Review at about 360px, 768px and 1280px, with keyboard-only navigation and a contrast check.
8. Run `scripts/validate-exercise.sh` if it works.

## Open questions

- **Launch type:** a `node-terminal` configuration is the likely fit for `serverReadyAction`; the Coder should pick the simplest option that works and prove it. `.vscode/tasks.json` already exists and should not be modified.
- **Vocabulary:** proposed statuses are Active / At Risk / Blocked / On Hold / Completed and priorities High / Medium / Low. Please confirm.
- **Sample content:** project data is invented placeholder content. Confirm that is acceptable.
- **Extras:** filtering, sorting and dark mode are out of scope, apart from an optional colour-scheme query.
