# Project Pulse Dashboard — Implementation Plan

## Summary

Project Pulse is a small, static web dashboard for Mona's contributor team. It presents a grid of project cards driven by a local `app/project-data.json` file, styled as a polished dashboard (cards, status badges, priority treatment, readable spacing, responsive layout), and previewable from VS Code via a **Run Project Pulse Dashboard** launch configuration.

The build is deliberately dependency-free: plain HTML, CSS, and a JSON data file loaded client-side. No framework, no build step, no package manager. The app is served as static files (`python3 -m http.server 5500` from `app/`) so `index.html` renders the dashboard frontend rather than a directory listing.

Four files are in scope, split across two specialists per repo conventions:

| File | Owner | Rationale |
| --- | --- | --- |
| `app/index.html` | **Coder** (structure/logic) with Designer input on markup semantics | Contains data-fetch logic and rendering structure; Coder owns behavior. Designer specifies semantic markup, headings, ARIA, and card structure. |
| `app/styles.css` | **Designer** | Pure UI/UX and visual design surface. |
| `app/project-data.json` | **Coder** | Data structure/config; deterministic schema. |
| `.vscode/launch.json` | **Coder** | Runnable-app support config; strict JSON, no comments. |

The plan matches the agent definitions: Designer owns UI/UX and CSS; Coder owns logic, structure, JSON, and launch config. It also aligns with the exact validation phrases the exercise checks (`.dashboard`, `.project-card`, `project-card`, `border-radius`, `box-shadow`, `index.html`, top-level `projects`, and the fields `name`, `owner`, `status`, `recentActivity`, `priority`).

---

## Ordered implementation steps

### Step 1 — Design direction and data contract (foundation)
Establish the shared contract before any file is written so Designer and Coder do not collide:
- Data schema: top-level `projects` array; each object has `name`, `owner`, `status`, `recentActivity`, `priority`.
- Agreed status vocabulary (e.g., `Active`, `At Risk`, `Blocked`, `Done`) and priority vocabulary (e.g., `High`, `Medium`, `Low`) so CSS badge classes and JSON values match.
- Required deterministic CSS hooks: `.dashboard` (page wrapper) and `.project-card` (each card).
- Markup contract: page `<h1>` contains the exact text `Project Pulse`; each card uses `class="project-card"` and renders `status`, `recentActivity`, and `priority`.
- **Owners:** Designer (visual/IA/accessibility decisions) + Coder (schema/field names). Produces shared decisions only; no competing file edits.

### Step 2 — Create the seed data file `app/project-data.json`
Author 4–6 representative projects using the agreed schema and status/priority vocabularies.
- **Owner:** Coder.

### Step 3 — Build the HTML structure and data-rendering logic `app/index.html`
- Exact `<h1>Project Pulse</h1>` title; `<meta charset>` and viewport meta.
- `<link rel="stylesheet" href="styles.css">`.
- A `.dashboard` container that holds project cards.
- Client-side logic that `fetch`es `project-data.json`, iterates `data.projects`, and renders one `.project-card` per project showing `name`, `owner`, `status`, `recentActivity`, and `priority`.
- Status/priority rendered with badge-friendly class hooks that match the CSS contract from Step 1.
- **Owner:** Coder (rendering logic + structure), applying Designer's semantic/accessibility guidance.

### Step 4 — Style the dashboard `app/styles.css` (parallel with Step 3)
- `.dashboard` layout (responsive grid, spacing).
- `.project-card` styling with `border-radius` and `box-shadow`.
- Status badges, priority treatment, typography, contrast, hover/focus affordances, responsive breakpoints.
- **Owner:** Designer.

### Step 5 — Create the launch configuration `.vscode/launch.json`
- Strict JSON, no comments.
- A configuration named exactly `Run Project Pulse Dashboard`.
- Serve from the app directory: `cwd` = `${workspaceFolder}/app`, command `python3 -m http.server 5500`.
- `serverReadyAction` that opens `http://localhost:%s/index.html` (so learners see the dashboard, not a directory listing).
- **Owner:** Coder.

### Step 6 — Integration and validation
Confirm the four files hang together, the page renders cards (not a directory listing), and validation criteria pass.
- **Owner:** Orchestrator coordinates; Coder and Designer verify their scopes.

---

## File assignments (with Designer vs Coder responsibilities)

### `app/index.html` — **Coder** (primary), Designer (advisory)
- **Coder responsibilities:** document structure; charset/viewport meta; `<link>` to `styles.css`; the fetch-and-render logic that reads `project-data.json` and builds `.project-card` elements for each project; exact `Project Pulse` title text; ensure `status`, `recentActivity`, and `priority` values appear in the rendered output; graceful handling of load/parse failure.
- **Designer responsibilities (advisory, no file edits here unless assigned):** semantic HTML (landmarks, heading order), accessible badge labeling, card content order/hierarchy, and which class hooks the CSS expects (`.dashboard`, `.project-card`, badge classes).

### `app/styles.css` — **Designer** (sole owner)
- **Designer responsibilities:** all visual design — `.dashboard` responsive grid, `.project-card` with `border-radius` and `box-shadow`, status/priority badge styling, typography, color/contrast, spacing, hover/focus states, responsive breakpoints. Polished dashboard, not a bare page.
- **Coder responsibilities:** none (do not edit this design-only file unless explicitly reassigned).

### `app/project-data.json` — **Coder** (sole owner)
- **Coder responsibilities:** valid strict JSON; top-level `projects` array; each project includes `name`, `owner`, `status`, `recentActivity`, `priority`; values use the agreed status/priority vocabulary; realistic contributor-friendly sample data.
- **Designer responsibilities:** advise on realistic sample content/tone only; does not edit the file.

### `.vscode/launch.json` — **Coder** (sole owner)
- **Coder responsibilities:** strict JSON with no comments; configuration named exactly `Run Project Pulse Dashboard`; `cwd` = `${workspaceFolder}/app`; `python3 -m http.server 5500`; `serverReadyAction` opening `http://localhost:%s/index.html`.
- **Designer responsibilities:** none.

---

## Dependencies between steps

- **Step 1 → everything.** The schema and CSS-hook contract must be agreed first; it prevents field-name and class-name mismatches.
- **Step 3 (index.html) depends on Step 2 (project-data.json):** the render logic consumes the JSON shape. If Step 3 starts before Step 2 finishes, it must rely on the Step 1 contract as the source of truth.
- **Step 4 (styles.css) depends on Step 1's class-hook contract**, not on the HTML file's internals — so it can proceed in parallel with Step 3 as long as both honor the agreed hooks.
- **Step 5 (launch.json)** depends only on the existence of `app/index.html` as the served entry point (path, not content). Can be authored anytime after the contract; best validated once `index.html` exists.
- **Step 6** depends on Steps 2–5 all being complete.

---

## Work that can run in parallel

- **Step 3 (`index.html`, Coder)** and **Step 4 (`styles.css`, Designer)** run in parallel — separate files, no overlapping scope, both bound by the Step 1 contract.
- **Step 5 (`.vscode/launch.json`, Coder)** can run in parallel with Step 4 (different owner, different file) and alongside Step 3 (same owner, so Coder sequences its own two files as capacity allows).
- **Step 2 (`project-data.json`, Coder)** can run in parallel with Step 4 (Designer).

Orchestrator note: parallel is safe because each file has exactly one editing owner and no two agents write the same file.

---

## Work that must run sequentially

- **Step 1 before all others** (shared contract).
- **Step 2 before final integration of Step 3**, since `index.html` renders the actual JSON. (Step 3 scaffolding may begin against the contract, but final verification requires the real data file.)
- **Step 6 (integration/validation) last**, after all four files exist.
- Because Coder owns three files (`index.html`, `project-data.json`, `launch.json`), Coder sequences its own work internally; recommended order: `project-data.json` → `index.html` → `launch.json`.

---

## Edge cases to handle

- **Directory listing instead of the app:** launch config must open `http://localhost:%s/index.html`, not `http://localhost:%s/`. Serving from `${workspaceFolder}/app` keeps paths relative and correct.
- **`fetch` blocked by `file://`:** loading `project-data.json` via `fetch` fails under `file://`. The app must be served over HTTP (the launch config handles this). Document that opening `index.html` directly from disk will not load data.
- **Relative path correctness:** `styles.css` and `project-data.json` are referenced relatively; serving from `app/` keeps them at the server root.
- **JSON parse/load failure:** `index.html` should show a visible, accessible error/empty state rather than a blank page if the fetch or parse fails.
- **Empty or missing `projects` array:** render a friendly empty state; do not crash.
- **Missing per-project fields:** render a safe fallback (e.g., "Unknown"/"—") instead of `undefined`.
- **Status/priority value drift:** JSON values must match the badge classes CSS styles; unmatched values should still render with a neutral default badge style.
- **Port already in use (5500):** note that the learner may need to stop a prior server; keep the port deterministic per the exercise.
- **Strict JSON:** both `.vscode/launch.json` and `app/project-data.json` must contain no comments and no trailing commas (`python3 -m json.tool` must parse them).
- **Accessibility:** sufficient color contrast on badges, focusable/readable cards, correct heading order, and non-color-only status cues.
- **Responsive layout:** grid must reflow cleanly on narrow viewports.

---

## Validation expectations

**`app/index.html`**
- Contains the exact text `Project Pulse`.
- Links to `styles.css`.
- References/loads `project-data.json`.
- Renders visible project cards using the `project-card` class.
- Displays `status`, `recentActivity`, and `priority` values.

**`app/styles.css`**
- Includes a `.dashboard` selector.
- Includes a `.project-card` selector.
- Includes `border-radius` and `box-shadow` (polished styling).

**`app/project-data.json`**
- Parses as valid JSON (`python3 -m json.tool app/project-data.json`).
- Has a top-level `projects` key (array).
- Each project includes `name`, `owner`, `status`, `recentActivity`, `priority`.

**`.vscode/launch.json`**
- Parses as strict JSON (`python3 -m json.tool .vscode/launch.json`).
- Contains a configuration named exactly `Run Project Pulse Dashboard`.
- Serves from the `app/` directory (`cwd` = `${workspaceFolder}/app`).
- Opens `http://localhost:%s/index.html`.

**Runtime check**
- Selecting **Run Project Pulse Dashboard** in Run and Debug opens the browser to `index.html` and shows the styled dashboard with project cards — not a directory listing. Stop the server after verifying.

---

## Open questions

1. **Exact status/priority vocabularies:** the plan proposes `Active/At Risk/Blocked/Done` and `High/Medium/Low`. Confirm the preferred set so badge CSS and JSON values align. *(Assumption made if unanswered.)*
2. **Number of seed projects:** proposing 4–6. Confirm the desired count/content tone. *(Assumption: 4–6.)*
3. **`last updated` field:** the task summary mentions "last updated," but the brief's required schema is `name/owner/status/recentActivity/priority`. Plan follows the brief's required fields; treating `recentActivity` as the recency signal. Confirm whether a separate `lastUpdated` field should also be added. *(Assumption: use `recentActivity` only, to match validation.)*
4. **Launch mechanism:** the exercise standardizes on `python3 -m http.server 5500` with `serverReadyAction`. Confirm `python3` is available in the environment (it is used by the repo's validation scripts, so this is expected).
5. **HTML editing ownership:** repo convention gives Coder logic/structure and Designer UI/UX. This plan assigns `index.html` to Coder with Designer advising on semantics/accessibility. Confirm the Orchestrator does not want Designer to directly edit `index.html` markup — if so, that must be a separate sequential phase to avoid two owners on one file.
