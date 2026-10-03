# Project Pulse Dashboard Implementation Plan

## Overview

Build a lightweight, static dashboard that helps Mona's contributors scan active
projects, owners, status, recent activity, priority or risk, and a short
contributor-friendly summary. The first view should be polished, accessible,
responsive, and clearly read as a dashboard. Use local JSON data and make the
VS Code **Run Project Pulse Dashboard** configuration open `app/index.html`
through a local HTTP server.

The source of truth for product requirements is
`.github/project-pulse-brief.md`; `.github/steps/3-step.md` and
`.github/workflows/3-step.yml` provide the learner-facing requirements and
automated checks. The repository currently has no `app/` files and no
`.vscode/launch.json`; `.vscode/tasks.json` is an existing Codespace task and
must remain untouched. No app framework, package manager, or build system is
present or needed. Keep the work to the four assigned implementation files.

## Agent responsibilities and file ownership

| Agent | Responsibility | Assigned files | Dependencies |
| --- | --- | --- | --- |
| Orchestrator | Coordinate the handoffs and phases, keep scopes disjoint, resolve blockers, and verify the integrated result. Do not implement the dashboard. | No implementation files | Coordinate after receiving the brief; wait for agent outputs before integration. |
| Designer | Own UX and accessible, responsive visual design; define the information hierarchy, visual affordances, and card system, then implement the agreed styling and CSS hooks. | `app/styles.css` | Depends on the agreed UX/CSS contract. |
| Coder | Own semantic page markup and data rendering, representative project data, and runnable preview configuration; validate app behavior. | `app/index.html` | Depends on the UX contract, CSS hooks, and JSON schema/data. |
| Coder | Own semantic page markup and data rendering, representative project data, and runnable preview configuration; validate app behavior. | `app/project-data.json` | Depends on the agreed data schema. |
| Coder | Own semantic page markup and data rendering, representative project data, and runnable preview configuration; validate app behavior. | `.vscode/launch.json` | Independent of app content; runtime validation depends on the app. |

Do not let the Designer edit markup or data, or the Coder edit the stylesheet.
The Orchestrator should communicate a small, explicit interface contract
(including `.dashboard` and `.project-card`) so the independently owned files
fit together.

## Ordered phases

### 1. Establish UX and implementation contracts

- **Lead:** Designer; Orchestrator coordinates. No files need to change.
- Agree on the first-view hierarchy and visible project fields; card and badge
  hooks; narrow-screen behavior; keyboard focus, contrast, and non-color status
  cues; and how a contributor-friendly summary will be presented.
- Preserve the required project data contract: a top-level `projects` array
  with each project's `name`, `owner`, `status`, `recentActivity`, and
  `priority`. Recommend an optional `summary` per project so the brief's
  contributor-friendly summary is authored rather than inferred.
- **Dependency:** Complete this agreement before parallel implementation begins.

### 2. Create styling and independent support files (parallel)

- **Designer — `app/styles.css`:** Implement the agreed dashboard/card layout,
  readable type and spacing, status badges, distinct priority/risk treatment,
  responsive behavior, visible focus states, and accessible contrast. Include
  `.dashboard` and `.project-card`; provide rounded cards and subtle shadows.
- **Coder — `app/project-data.json`:** Add valid JSON with a top-level
  `projects` array and multiple deterministic, representative records. Include
  all five required fields and the agreed optional summary. Keep values
  consistent with the labels/treatments in the UX contract.
- **Coder — `.vscode/launch.json`:** Add strict JSON and a configuration named
  **Run Project Pulse Dashboard**. Use `python3 -m http.server 5500`, set
  `cwd` to `${workspaceFolder}/app`, and configure `serverReadyAction` to open
  `http://localhost:%s/index.html` externally when the server is ready. Ensure
  the chosen VS Code launch type supports that command and readiness action.
- **Parallel decision:** The stylesheet, data, and launch configuration have
  distinct owners and can be created concurrently once the interface and data
  contracts are agreed. They do not depend on one another's file contents.

### 3. Implement and connect the page (sequential)

- **Owner:** Coder; use Designer's agreed hooks and styling contract.
- **File:** `app/index.html`.
- Create a semantic document with the exact title **Project Pulse**, link
  `styles.css`, and load `project-data.json` over HTTP. Render a visible
  `.project-card` per project, presenting name, owner, status, recent activity,
  priority, and summary. Match the agreed responsive markup and accessible
  labels.
- Show a clear empty state for an empty `projects` array and a useful error
  state if loading or parsing data fails; do not leave a blank view or an
  unhandled rejection.
- **Dependency:** Start after Phase 1's contract and Phase 2's CSS hooks and
  data shape are settled. This ordering avoids class/field mismatches while
  keeping the CSS, data, and launch work parallel.

### 4. Integrate, validate, and hand off (sequential)

- **Coordinator:** Orchestrator. **Validation owners:** Coder for structure,
  JSON, launch, and runtime checks; Designer for visual/accessibility review.
- Review all four assigned files together. Ask the owning agent to fix any
  integration defect; do not broaden file ownership.
- Run **Run Project Pulse Dashboard** from VS Code, verify it serves the
  `app/` directory and opens the dashboard page, inspect the browser and
  console, then stop the server.
- Report what was validated and any environment limitation. Do not stage,
  commit, or push.
- **Dependency:** Requires Phases 2 and 3 to be complete.

## Dependencies and parallelism summary

1. The UX, class-hook, and data-shape agreement is a prerequisite for all
   implementation.
2. After that agreement, Designer's `app/styles.css` and Coder's
   `app/project-data.json` plus `.vscode/launch.json` can be produced in
   parallel; file scopes are separate.
3. Coder's `app/index.html` follows the contract and initial CSS/data outputs,
   so page integration is sequential rather than an overlapping parallel task.
4. Integrated browser/launch validation follows all four files. Static
   validation of JSON and required strings can run independently within this
   final phase, but end-to-end verification cannot.

## Parallel work decisions

- **Phase 1 is sequential prerequisite work:** agree on the UX, interface/CSS
  hooks, and JSON schema/data contract before implementation starts.
- **Parallel after Phase 1:** Designer can build `app/styles.css` while Coder
  creates `app/project-data.json` and `.vscode/launch.json`; these file scopes
  are independent.
- **Page implementation waits:** Coder must begin `app/index.html` only after
  the agreed contract, CSS hooks, and JSON schema/data are available.
- **Final validation follows implementation:** integrate and run end-to-end
  validation after all implementation files are complete. Static checks can
  run in parallel during this final phase; end-to-end validation cannot.

## Edge cases and risks

- `fetch()` is not reliable when the page is opened directly using `file://`;
  validate via the configured HTTP server.
- Port 5500 may be occupied. If so, identify the conflict and coordinate any
  port change across the server command and URL rather than changing only one.
- VS Code's launch support for the server command and `serverReadyAction` may
  depend on the available debugger/extension. Verify in the target Codespace
  and report a missing prerequisite instead of silently changing the required
  launch behavior.
- Empty, unavailable, or malformed project data must produce understandable
  feedback. Unknown statuses/priorities should remain readable and should not
  rely on color alone.
- Long names/activity/summaries and small viewports can cause overflow or
  unreadable cards; confirm text wraps and layout adapts.
- The five required JSON fields omit summary. Keep them mandatory and use an
  optional `summary` field for the requested extra copy; do not make summary
  inclusion a reason to omit any required field.

## Validation expectations

- Confirm `app/index.html`, `app/styles.css`, `app/project-data.json`, and
  `.vscode/launch.json` exist.
- Parse both JSON files. Confirm `project-data.json` has a top-level array
  named `projects`, and every project has non-empty `name`, `owner`, `status`,
  `recentActivity`, and `priority`.
- Confirm the HTML includes the exact **Project Pulse** title, references
  `styles.css` and `project-data.json`, uses `.project-card`, and renders all
  required project fields from the data. Check explicit empty and failure
  states.
- Confirm CSS includes `.dashboard`, `.project-card`, responsive layout,
  `border-radius`, and `box-shadow`. Visually review hierarchy, text
  readability, status/priority distinction, keyboard focus, contrast, and a
  narrow viewport.
- Confirm launch JSON is strict JSON, has the exact configuration name,
  command `python3 -m http.server 5500`, `cwd` `${workspaceFolder}/app`, and a
  readiness action that opens `http://localhost:%s/index.html`.
- In VS Code, run the named configuration. Verify the browser shows the
  dashboard rather than a directory listing; data loads without network or
  console errors; cards appear; and the server can be stopped cleanly.
- Temporarily test empty data and a missing/invalid JSON response, then restore
  valid sample data.
- `.github/workflows/3-step.yml` checks required files and key phrases and
  parses the two JSON files. It does not launch a server or check runtime or
  visual behavior, so those still need manual verification. `scripts/validate-exercise.sh`
  validates template/exercise conventions rather than the dashboard itself.

## Open questions

- Are actual project names, owners, activity, and priority values available,
  or should the initial data use clearly representative examples?
- Confirm the optional per-project `summary` field is acceptable for the
  contributor-friendly copy; the brief requires that copy but does not specify
  its data shape.
- Is the VS Code launch/debug support needed for `serverReadyAction` available
  in the target Codespace? If not, what compatible launch type is supported
  while preserving the required command, app working directory, and browser
  URL?
