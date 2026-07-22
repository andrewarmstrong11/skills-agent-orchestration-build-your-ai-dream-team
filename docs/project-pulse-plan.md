# Implementation Plan: Project Pulse Dashboard

*(To be saved by the Orchestrator to `docs/project-pulse-plan.md`)*

---

## 1. Summary

**Project Pulse** is a small, static, read-only dashboard that helps Mona's team and contributors quickly answer: *which projects are active, who owns them, what's their current status, what happened recently, and how urgent/risky are they?*

Goals:
- A single-page static app (no build step, no framework, no backend) consisting of exactly three app files: `app/index.html`, `app/styles.css`, `app/project-data.json`.
- The dashboard renders a **grid of project cards** sourced entirely from `app/project-data.json`, using client-side JavaScript embedded in (or referenced from) `app/index.html`.
- Visual polish is required — this must look like a real dashboard product, not a bare unstyled page: card layout, status badges, priority treatment, rounded corners, shadows, responsive behavior.
- A learner previewing this in a Codespace must be able to launch it with one click via a VS Code **Run and Debug** configuration (`.vscode/launch.json`) named **"Run Project Pulse Dashboard"**, which serves `app/` and opens `index.html` directly (not a directory listing).
- This matches the repository's existing exercise structure (`.github/project-pulse-brief.md`, `.github/steps/2-step.md`, `.github/steps/3-step.md`), which already fixes several implementation details as hard requirements (see below). This plan conforms to those fixed requirements rather than inventing new ones, since a validation script (`scripts/validate-exercise.sh`) and workflow keyphrase checks enforce them.

**Hard requirements already fixed by the repo (must be honored exactly):**
- `app/index.html` must contain the exact title text **"Project Pulse"**, must reference `styles.css` and `project-data.json`, and must render project cards using the class **`project-card`**, displaying each project's `status`, `recentActivity`, and `priority`.
- `app/styles.css` must include a **`.dashboard`** selector and a **`.project-card`** selector, and must use **`border-radius`** and **`box-shadow`** somewhere (polish signals checked by validation).
- `app/project-data.json` must have a top-level **`"projects"`** array/key, and each project object must include **`name`, `owner`, `status`, `recentActivity`, `priority`** (exact field names).
- `.vscode/launch.json` must be strict JSON (no comments), define a launch configuration named exactly **"Run Project Pulse Dashboard"**, serve from the `app/` directory (`cwd`: `${workspaceFolder}/app`), run `python3 -m http.server 5500`, and use a `serverReadyAction` that opens `http://localhost:%s/index.html` — not a directory listing.

No `app/` or `.vscode/launch.json` currently exist in the repository, so both must be created from scratch. There is an existing `.vscode/tasks.json` but no `launch.json`, so this will be a net-new file, not a merge.

---

## 2. Ordered Implementation Steps

| # | Step | Primary Owner |
|---|------|----------------|
| 1 | Define the data contract: finalize the shape of `app/project-data.json` (fields, value domains for `status`/`priority`, sample project count) | Coder (data owner), informed by Designer's needs for badge/priority styling |
| 2 | Draft the information architecture and visual direction for the dashboard (layout, card anatomy, badge system, responsive breakpoints, CSS hooks) | Designer |
| 3 | Build the semantic HTML structure in `app/index.html` (title, dashboard container, script tag, fetch/render logic, empty-state handling) | Coder |
| 4 | Apply visual styling in `app/styles.css` against the agreed CSS hooks (`.dashboard`, `.project-card`, badge classes, responsive grid) | Designer |
| 5 | Wire `app/index.html` rendering logic to consume `app/project-data.json` end-to-end (fetch, loop, inject fields, badge classes matching styles.css) | Coder |
| 6 | Create `.vscode/launch.json` runnable configuration | Coder |
| 7 | Joint review pass: Coder + Designer verify the rendered dashboard matches design intent and data contract | Coder & Designer |
| 8 | Validation pass (JSON validation, launch test, accessibility/responsive spot-check) | Coder (technical validation) + Designer (UX/accessibility validation) |

---

## 3. File Assignments

### `app/project-data.json`
- **Owner:** Coder
- **Content:** Top-level `"projects"` array. Each object: `name` (string), `owner` (string), `status` (string; recommend a small closed set such as `"On Track"`, `"At Risk"`, `"Blocked"`, `"Complete"` — but implementation may vary status text as long as it's a string that maps to a badge style), `recentActivity` (short string, e.g. "Merged auth refactor PR"), `priority` (string; recommend `"High"`, `"Medium"`, `"Low"`).
- Include at least 4–6 sample projects to make the grid/responsive layout visually meaningful, including at least one long project name and one long `recentActivity` string to stress-test layout.
- Must remain valid JSON (no trailing commas, no comments) — validated with `python3 -m json.tool`.

### `app/index.html`
- **Owner:** Coder (structure/logic), with structural/markup guidance from Designer (semantic elements, ARIA roles, heading hierarchy).
- Must include the literal page/title text **"Project Pulse"**.
- Must `<link>` to `styles.css` and reference `project-data.json` (via `fetch()` — static file served over `http.server`, so `fetch('./project-data.json')` will work; no CORS issue since served from same origin).
- Must contain a top-level container element with class **`dashboard`** and, for each rendered project, an element with class **`project-card`**.
- Each card must visibly show `status`, `recentActivity`, and `priority` (plus `name` and `owner` per the brief).
- Rendering must happen via inline `<script>` (kept in `index.html` for simplicity, matching "small static app" scope — no separate `app.js` file was requested in file assignments, so keep script inline unless Orchestrator explicitly expands scope).
- Must handle the empty/missing-data case gracefully (see Edge Cases).

### `app/styles.css`
- **Owner:** Designer
- Must include a **`.dashboard`** selector (grid/flex container, responsive columns, page background/typography baseline) and a **`.project-card`** selector (border-radius, box-shadow, padding, spacing).
- Must define visually distinct status-badge and priority treatments (e.g. `.status-badge`, `.status-on-track`, `.priority-high`, etc.) with color + icon/text differentiation, not color alone (accessibility).
- Must include `border-radius` and `box-shadow` literally somewhere (validation checks for these keywords).
- Must include responsive rules (e.g. `@media` breakpoint or CSS Grid `auto-fit`/`minmax`) so the card grid reflows on narrow viewports.

### `.vscode/launch.json`
- **Owner:** Coder
- Strict JSON, no comments.
- One configuration:
  - `name`: exactly `"Run Project Pulse Dashboard"`
  - `type`: `"node-terminal"` (the only built-in VS Code debug type that supports running an arbitrary shell command with `serverReadyAction` pattern matching against terminal output; recommended since the run command is a Python one-liner, not a Node app)
  - `request`: `"launch"`
  - `command`: `"python3 -m http.server 5500"`
  - `cwd`: `"${workspaceFolder}/app"`
  - `serverReadyAction`: `{ "pattern": "Serving HTTP on .* port ([0-9]+)", "uriFormat": "http://localhost:%s/index.html", "action": "openExternally" }`
- This exact pattern matches Python's `http.server` startup log line ("Serving HTTP on 0.0.0.0 port 5500 …") to capture the port and open `index.html` directly instead of a bare directory listing.
- Validate with `python3 -m json.tool .vscode/launch.json`.

---

## 4. Designer Responsibilities

- Own `app/styles.css` end-to-end.
- Define and communicate the **CSS hook contract** to Coder *before* Coder finalizes markup: class names `.dashboard`, `.project-card`, plus any badge/priority class names (e.g. `.status-badge`, `.priority-high/medium/low`) so Coder's HTML emits matching classes.
- Specify information hierarchy inside each card: project name as heading, owner as secondary text, status as a colored badge, priority as a distinct visual treatment (badge, colored border-left, icon), recent activity as supporting text.
- Specify responsive behavior: how many columns at desktop vs. tablet vs. mobile widths; how cards reflow or stack.
- Specify accessibility requirements for Coder's markup: semantic landmarks (`<main>`, `<h1>`, `<h2>` per card), sufficient color contrast for badges, not relying on color alone to convey status/priority (add text labels/icons), focus-visible states if any interactive elements exist.
- Review the final rendered dashboard visually (open via the launch config) and confirm it "looks like a Project Pulse dashboard," not a bare page.
- Does not touch `app/project-data.json` or `.vscode/launch.json`.
- May propose (but not implement) structural changes to `app/index.html`; any HTML edits should be handed back to Coder unless Orchestrator explicitly grants Designer write access to `index.html` for a specific fix, to avoid file-scope conflicts.

## 5. Coder Responsibilities

- Own `app/index.html` (structure + rendering logic), `app/project-data.json`, and `.vscode/launch.json`.
- Finalize the data schema in `project-data.json` early (Step 1) so Designer can style against real field names/values.
- Build semantic, accessible HTML skeleton: `<h1>Project Pulse</h1>`, a `<main class="dashboard">` container, and a template/loop that Coder implements to inject one `.project-card` element per project.
- Implement the fetch/render logic: `fetch('./project-data.json')`, parse `data.projects`, loop and build DOM nodes (or template-string HTML) with `name`, `owner`, `status`, `recentActivity`, `priority`, applying Designer's class-naming contract for badges.
- Implement `.vscode/launch.json` exactly per the spec above; smoke-test it by running the configuration in the Codespace before reporting completion.
- Validate JSON validity of `project-data.json` and `launch.json` with `python3 -m json.tool`.
- Does not modify `app/styles.css` beyond adding/removing class names on elements it owns (visual rule authorship stays with Designer).

---

## 6. Dependencies Between Steps

1. **Data shape before rendering:** `app/project-data.json`'s field names/values must be finalized before Coder writes the render loop in `index.html`, and before Designer finalizes badge class names tied to `status`/`priority` values.
2. **CSS hook contract before final markup polish:** Designer must communicate class names (`.dashboard`, `.project-card`, badge classes) before Coder finalizes the exact `class` attributes emitted in the render loop — though Coder can build a functional first pass with placeholder classes in parallel (see Section 7).
3. **`index.html` existing before `launch.json` is meaningful:** `.vscode/launch.json` only needs `app/` to exist as a directory with an `index.html` entry point to be testable; Coder should create a minimal `index.html` skeleton early so the launch config can be smoke-tested even before full styling/rendering logic lands.
4. **Styling depends on markup existing:** Designer cannot finish `.project-card` styles until Coder's HTML actually emits `.project-card` elements (even a static/hardcoded sample card is enough to unblock Designer — Designer does not need to wait for the dynamic fetch/render logic to be wired).
5. **Final validation depends on all four files being complete and mutually consistent** (Step 8 depends on Steps 1–6 and the joint review in Step 7).

---

## 7. Parallel Work Decisions

**Can run in parallel (no file conflicts):**
- Designer working on `app/styles.css` visual rules **once a class-name contract and at least one sample static card markup exist** — Designer can iterate purely in the CSS file while Coder simultaneously wires up the dynamic fetch/render logic and finalizes `project-data.json`, as long as both are working from the same agreed class-name contract.
- Coder finalizing `app/project-data.json` content/schema in parallel with Designer drafting layout direction/wireframe notes (Designer doesn't need the final data values, just the field names and rough value domains for status/priority, to design badges).
- Coder creating `.vscode/launch.json` can happen at any point once a minimal `app/index.html` exists as a file (does not need to wait for full styling or data wiring to be complete).

**Must be sequential / require handoff:**
- Step 1 (data schema) → Step 3 (HTML render logic) and → Step 2/4 (Designer badge styling) — schema must be agreed first.
- Step 2 (Designer's class-name contract) → final class attributes in Step 3/5 (Coder's markup) — avoid Designer and Coder simultaneously renaming the same classes independently.
- Step 7 (joint review) must happen only after both Steps 3–6 are functionally complete — this is a merge/consistency checkpoint, not something to parallelize.
- Step 8 (validation) is strictly after Step 7.

**Orchestrator phasing recommendation:**
- **Phase A (sequential, short):** Coder drafts `project-data.json` schema + a couple of sample entries; Designer reviews field names and proposes badge/priority class names. (Quick handoff, not full parallel work.)
- **Phase B (parallel):** Coder builds `index.html` skeleton + render logic + `launch.json`; Designer builds out `styles.css` against the agreed class contract and a static reference card markup snippet.
- **Phase C (sequential):** Joint review — Coder wires final classes into the live render loop to match Designer's CSS; Designer confirms the live rendered output.
- **Phase D (sequential):** Validation.

---

## 8. Edge Cases to Handle

- **Empty project list** (`"projects": []`): dashboard should show a clear empty-state message inside `.dashboard`, not a blank page or JS error.
- **Missing/malformed JSON** (fetch fails, 404, invalid JSON): Coder's script should catch the fetch/parse error and render a visible error message rather than failing silently or throwing an uncaught exception in the console.
- **Missing individual fields** on a project object (e.g. no `priority`): render loop should not throw; should render a fallback (e.g. "Unknown") rather than `undefined`.
- **Long project names / long `recentActivity` strings**: styles must wrap text gracefully (`overflow-wrap`/`word-break`) rather than overflowing the card or breaking the grid.
- **Many distinct `status`/`priority` values** beyond the recommended set: badge CSS should have a sensible default/fallback style for unrecognized status strings so new statuses don't render unstyled.
- **Large number of projects**: grid should remain usable/scrollable, not require horizontal scrolling.
- **Accessibility**: color must not be the only signal for status/priority (include text or icon); heading hierarchy must be logical (`h1` once, `h2`/`h3` per card); sufficient contrast ratios on badges; cards should be reachable/readable by screen readers (avoid `div`-only soup without any semantic/ARIA labeling for card fields).
- **Responsive behavior**: verify card grid reflows correctly at common breakpoints (mobile ~375px, tablet ~768px, desktop ~1280px) without overlap or clipping.
- **Launch configuration collisions**: port 5500 must not conflict with any other running preview task in the Codespace; if the port is already in use, the learner will see a clear Python error in the terminal — Coder should note this as a known limitation rather than silently failing.
- **`serverReadyAction` pattern mismatch**: if Python's log format ever changes across versions/locales, the regex pattern may fail to match — Coder should test the actual printed log line in the Codespace's Python version before finalizing the pattern.

---

## 9. Validation Expectations

- **JSON validity:** `python3 -m json.tool app/project-data.json` and `python3 -m json.tool .vscode/launch.json` must both succeed without errors.
- **Static keyword/structure checks** (mirroring the repo's own validation approach in `scripts/validate-exercise.sh` and `.github/workflows/3-step.yml`):
  - `app/index.html` contains `Project Pulse`, references `styles.css` and `project-data.json`, and contains `project-card`.
  - `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius`, `box-shadow`.
  - `app/project-data.json` contains a top-level `projects` key and per-project `name`, `owner`, `status`, `recentActivity`, `priority` fields.
  - `.vscode/launch.json` contains `Run Project Pulse Dashboard`, references `app` directory (`cwd`), and opens `http://localhost:%s/index.html`.
- **Functional/manual validation (in the Codespace):**
  1. Open **Run and Debug** in VS Code, select **"Run Project Pulse Dashboard"**, press play.
  2. Confirm a browser tab/preview opens directly to `index.html` (dashboard UI), not a raw file/directory listing.
  3. Visually confirm: page title/heading reads "Project Pulse"; multiple project cards are visible; each card shows name, owner, status badge, priority indicator, and recent activity text; cards have rounded corners and shadows.
  4. Resize the browser/preview pane (or use responsive device toolbar) to confirm the grid reflows at mobile widths without horizontal scrolling or overlapping content.
  5. Temporarily edit `project-data.json` to an empty `projects` array (in a scratch/test, not committed) to confirm the empty-state message renders correctly, then revert.
  6. Stop the preview server (per the step instructions) before considering validation complete.
- **Accessibility spot-check:** tab through the page (if any interactive elements exist), confirm heading levels are logical via browser dev tools accessibility tree, and confirm status/priority are conveyed with text/icons in addition to color.

---

## 10. Open Questions

- Should `status` and `priority` be constrained to a fixed enum (e.g., exactly `"On Track" | "At Risk" | "Blocked" | "Complete"` and `"High" | "Medium" | "Low"`), or left as free-form strings with a generic fallback badge style? The brief doesn't fix exact values, only field names — Coder and Designer should agree on a small canonical set to keep badge styling deterministic, but this isn't validated strictly by the repo's automated checks, so any consistent choice is acceptable.
- Should the render/fetch script live inline in `app/index.html`, or should the Orchestrator explicitly authorize a separate `app/app.js` for cleaner separation of concerns? The brief and file-assignment list only mention three app files plus `launch.json`, so the default assumption in this plan is **inline script**, keeping strictly to the four assigned files unless the Orchestrator expands scope.
- Is `port 5500` guaranteed free in every learner's Codespace, or should the launch config/plan account for port-already-in-use fallback guidance? The repo's step file fixes `5500` explicitly, so this plan follows that, but learners hitting a port conflict may need manual troubleshooting notes (out of scope for this plan, but worth flagging to the Orchestrator/learner).
- Should Designer be granted narrow write access to `app/index.html` for purely cosmetic class-name additions to avoid an extra handoff round-trip, or should all HTML edits strictly flow through Coder to keep file-ownership boundaries clean? This plan recommends the stricter boundary (Designer proposes, Coder implements) to avoid concurrent-edit conflicts, but the Orchestrator may relax this if turnaround time is a higher priority than strict ownership separation.
