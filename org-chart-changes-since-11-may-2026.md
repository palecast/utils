# Org Chart — Changes since 11 May 2026

This document describes the changes made to `org-chart.html` between 11 May and 12 June 2026 — 9 commits in total, all landing between 3 June and 12 June. New features and behaviours are grouped thematically; defects and minor items are listed at the end.

---

## 1. Form-based editing and the hierarchical manager picker *(11 June)*

Two changes lifted editing out of the table grid into something safer and more deliberate.

### Role editor dialog
A single dialog now edits every field of a role at once. It opens from a row edit button on the Table tab, by double-clicking a grid row or a chart node, or from the chart's right-click menu. A save is recorded as one atomic undo step, and Role ID renames are validated before being applied (and correctly propagated to reports).

### Hierarchical manager picker
Changing a role's manager — both inside the editor dialog and directly on the grid — now opens a tree popup mirroring the org hierarchy, the same style as the Chart tab's "Show from" picker. It supports search, auto-expands the path to the current manager, and excludes the role itself plus all its descendants so a cycle cannot be created by construction.

**Benefit:** Multi-field changes become one atomic, undoable action rather than a series of brittle cell edits, and structural mistakes (reporting cycles) are impossible by construction.
**Use cases:** Onboarding a new hire with every field filled in one dialog; restructuring a team mid-meeting by picking the new manager off the actual tree; safely experimenting with "what if this team moved under Ops?" and reverting in one undo.

---

## 2. Search and navigation

### Fit-to-view and zoom hotkeys *(3 June)*
A fourth View button scales and recentres the chart so the whole visible structure fits the canvas. Chart-tab-scoped hotkeys cover every zoom action: `+` / `-` to zoom, `0` to reset, `1` to fit-to-view, each surfaced in its button's tooltip.

### Find box on the chart tab *(12 June)*
A search box on the Chart tab matches against Role ID, job title, incumbent name, and department. Repeated searches cycle through matches; for each hit the chart expands any collapsed ancestors and pans the node into view.

**Benefit:** Locating a specific person never requires manually expanding branches, and moving between "whole org" and "one branch" views is one keystroke.
**Use cases:** Presenting (fit the whole org, then dive into a branch); "where does Priya sit?" answered in two keystrokes; cycling between several people with the same surname.

---

## 3. Colour key (legend) rework *(4–5 June)*

The on-screen colour key was reworked across two commits, then realigned with exports a day later:

- **Horizontal bottom strip** *(4 June)* — the legend lays out horizontally along the bottom of the chart as a floating overlay, instead of a vertical box in the top-right, reclaiming canvas space.
- **Distinct colours** *(4 June)* — the cycling 10-colour palette was replaced with a curated 12-colour set plus a golden-angle fallback generator, so two categories never share a colour no matter how many values the chosen dimension has.
- **Draggable** *(4 June)* — the legend can be dragged anywhere over the chart, and snaps back to its default lower-left corner whenever the Colour dimension changes.
- **Matching exports** *(5 June)* — Print and PNG output place the colour key as a centred horizontal strip at the bottom, matching the on-screen presentation.

**Benefit:** The legend no longer obscures the chart, never produces ambiguous duplicate colours, and what you see matches what you print or share as PNG.
**Use cases:** Colouring by a high-cardinality dimension (many departments) without colour collisions; dragging the key out of the way mid-presentation; printed charts that recipients can interpret without the app.

---

## 4. Comparing organisational snapshots *(12 June)*

A Compare… button in the header diffs the currently loaded data against a second CSV/XLSX file. The result classifies every role as added, removed, moved (manager changed), or changed (field edits), and reports headcount and cost deltas. The full diff can be exported as CSV.

**Benefit:** Turns the tool from a viewer/editor into a change-analysis tool — you can quantify exactly what differs between two versions of the org.
**Use cases:** Month-over-month org review against the latest HR extract; validating that a planned reorg file contains only the intended moves; an auditable change list with cost impact for leadership sign-off.

---

## 5. Crash-recovery autosave and multi-tab safety *(12 June)*

### Per-tab autosave snapshots
Unsaved working data is continuously snapshotted (debounced) to the browser's `localStorage`. If the tab is closed or crashes before saving, the next visit shows a restore banner offering to bring the work back. The snapshot is cleared whenever you save or open a file, so it never offers stale data.

Snapshots are scoped per tab (`orgChartAutosave:<tabId>`, with the tab id kept in `sessionStorage` so it survives reload) and a companion heartbeat key is refreshed every 5 s while the tab is dirty. Restore is offered only for snapshots whose owning tab is dead (stale heartbeat), with the exception of the tab's own snapshot, which is always offered after a reload. Discard surfaces the next orphan if several tabs crashed; snapshots older than 7 days and orphaned heartbeat keys are garbage-collected.

### Multi-tab warning
Tabs announce the file they have open over a `BroadcastChannel`. If two tabs hold the same file name, each shows a dismissible amber warning that edits are not shared and saves overwrite each other. The warning clears when either tab closes or moves to a different file.

**Benefit:** The full edit–save–reopen lifecycle of a desktop app, with a safety net underneath it: losing the tab no longer loses the session, and two tabs editing the same file no longer silently fork the document or clobber each other's recovery snapshot.
**Use cases:** Long restructuring sessions surviving a browser crash or accidental tab close; iterating on an org design across multiple sittings; noticing immediately when you've opened the same plan twice instead of merging conflicting saves later.

---

## 6. Exports

### Filter-aware CSV/Excel *(11 June)*
Export CSV/Excel now exports **what the active tab is showing**, not the raw dataset: grid header filters on the Table tab; filter clauses, tags, and the "Show from" subtree on the Chart and Pivot tabs. Where a role's manager is filtered out, the manager link is rewritten to the nearest visible ancestor so the exported file is still a coherent hierarchy.

### Field picker on CSV/Excel export *(12 June)*
Export CSV/Excel now opens a dialog to pick which columns to include. **Cost to Company is deliberately unticked by default**, so salary data only leaves the tool by explicit choice; Role ID is always included so exports re-import cleanly. The selection is remembered for the session. Save / Save As are unchanged and always write every field.

### SVG export *(12 June)*
The chart can be exported as SVG (rendered via `foreignObject` with the styling embedded), alongside PNG and Print.

### Pivot export matches the pivot *(12 June)*
The Pivot tab exports the rendered cross-tab (the aggregated table you see) instead of the underlying raw rows.

**Benefit:** What you see is what you share — exports are scoped to the visible slice, available in a vector format that stays crisp at any size, and salary data does not leak in a casual export.
**Use cases:** Sending a department head just their own subtree without leaking the wider org's salary data; embedding crisp SVG in slide decks; pasting a headcount cross-tab straight into a monthly report; emailing a re-importable plan to a peer without re-typing column choices each time.

---

## Defects fixed

- **Grid edits bypassed undo** *(11 June)* — cell edits never pushed undo snapshots (and Role ID renames never propagated to children) because Tabulator mutates row data before `cellEdited` fires; grid edits are now properly undoable.
- **Dialogs opened in the top-left corner** *(11 June)* — the global CSS margin reset was wiping the browser's native `<dialog>` centring; all dialogs now open centred.
- **Manual collapse lost on re-render** *(5 June)* — manually-collapsed branches are now preserved across drag-and-drop edits and across layout and colour changes (collapsed Role IDs are tracked and reapplied after each render). Collapses are intentionally cleared when the Depth control, filters, or the "Show from" focus changes.
- **Manual collapse not recorded in top-down layout** *(6 June)* — in the default top-down layout the children-collapse control is dabeng's `.bottomEdge`, not `.toggleBtn`; the tracking listener only matched `.toggleBtn`, so edge-driven collapses were never recorded and were lost on the post-drop re-render. The listener now also handles directional edges, and the dotted-line overlay redraws correctly on edge-driven collapses too.
- **Legend hidden or duplicated** *(5 June)* — a duplicate legend element was removed and the legend lifted above the fixed Summary panel, so it is no longer hidden behind the Summary view or sidebar after a re-draw.

## Other minor items

- **Tag tokenisation refined** *(11 June)* — tag filters split tags on whitespace **or** commas and match whole words.
- **Toasts on invalid dotted-manager IDs** *(12 June)* — invalid dotted-line manager IDs found during import are scrubbed with a toast notification explaining which IDs were dropped and why.
- **Dead layout presets removed** *(12 June)* — the unused `top` / `left` layout presets were deleted; the top-down hybrid is the single supported layout (the selector itself was dropped on 3 May).
