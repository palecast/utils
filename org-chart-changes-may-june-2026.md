# Org Chart — What's New (1 May – 12 June 2026)

This document describes the changes made to `org-chart.html` between 1 May and 12 June 2026 — 23 commits in total. New features and behaviours are grouped thematically; defects and minor items are listed at the end.

---

## 1. New rendering engine and hybrid layout *(1 May)*

The chart was migrated from the SVG-based d3-org-chart library to the DOM-based dabeng/OrgChart. Because the chart is now ordinary HTML, native browser printing and html2canvas PNG export render it faithfully instead of approximating it.

The migration introduced a **Top-down Hybrid layout**: any manager whose visible children are all leaves (under the current depth limit) stacks those children vertically, while the upper levels remain horizontal. The rule is depth-aware and re-resolves whenever the Depth selector changes.

A follow-up (*8 May*) added a **2-column wide-vertical layout**: a manager with more than six reports that are all leaves lays them out as a two-column grid with a centre spine and horizontal stubs, rather than one very tall column.

**Benefit:** Large flat teams (a manager with 15 individual contributors) no longer blow out the chart horizontally or vertically; exports look exactly like the screen.
**Use cases:** Charts mixing deep management chains with wide frontline teams; print/PNG output good enough to circulate without manual cleanup.

---

## 2. Working with files like real documents

### Open / Save / Save As *(1 May)*
File handling now models a real "open document": Open, Save, and Save As buttons with a filename indicator in the header. In Chromium browsers the File System Access API enables true in-place save to the original file; Firefox/Safari fall back to download.

### Unsaved-changes indicator *(1 May)*
A leading "•" appears on the filename whenever in-memory data has diverged from the saved file, clearing on save/open. Only genuine data mutations trigger it — display-only operations (filters, colour dimension, search, tab switches) deliberately do not.

### Crash-recovery autosave *(12 June)*
Unsaved working data is continuously snapshotted (debounced) to the browser's localStorage. If the tab is closed or crashes before saving, the next visit shows a restore banner offering to bring the work back. The snapshot is cleared when you save or open a file, so it never offers stale data.

**Benefit:** The full edit–save–reopen lifecycle of a desktop app, with a safety net underneath it: you always know whether your work is saved, and losing the tab no longer loses the session.
**Use cases:** Iterating on an org design across multiple sittings; long restructuring sessions surviving a browser crash or accidental tab close.

---

## 3. Editing the organisation

### Chart-side editing: right-click menu and drag-and-drop *(6 May)*
The chart canvas, previously read-only, became directly editable:

- **Right-click any role** to add a child, add a sibling, rename, jump to that row in the table, or delete.
- **Drag a node onto another** to change its manager. Drops that would create a reporting cycle (onto the node itself or any descendant) are refused; moving a manager together with their descendants prompts a confirmation dialog.

### Undo and Redo *(6–7 May)*
Cmd/Ctrl+Z undoes the last structural change (rename, add, delete, drag-move), with a paired redo stack on Cmd/Ctrl+Shift+Z. Both are also surfaced as buttons in the sidebar's View section, disabled when their stack is empty. Undo is suspended while typing in table cells so Tabulator's own cell-level undo is untouched.

### Form-based role editor *(11 June)*
A dialog edits every field of a role in one place. It opens from a row edit button on the Table tab, by double-clicking a grid row or a chart node, or from the chart's right-click menu. A save is recorded as a single undo step, and Role ID renames are validated before being applied (and correctly propagated to reports).

### Hierarchical manager picker *(11 June)*
Changing a role's manager in the grid opens a tree popup mirroring the org hierarchy — the same style as the Chart tab's "Show from" picker — with search, the path to the current manager auto-expanded, and the role itself plus all its descendants excluded so a cycle cannot be created.

**Benefit:** Restructuring happens where you can see it — on the chart — with structural mistakes (cycles) impossible by construction and every step reversible. The form editor makes multi-field changes one atomic, undoable action instead of a series of cell edits.
**Use cases:** Sketching a reorg live in a leadership meeting by dragging teams around; onboarding a new hire with all fields filled in one dialog; safely experimenting ("what if this team moved under Ops?") and undoing.

---

## 4. Dotted-line reporting *(8 May)*

Roles carry an optional list of dotted-line (secondary) managers, surfaced three ways: as dashed pills on each chart card, as a column in the table, and — via an opt-in sidebar toggle — as a sage-green dashed overlay drawn between the linked nodes on the chart. Dotted links are added and removed from the right-click menu through a searchable role picker. The new column round-trips through CSV/Excel, and validation scrubs self-loops, references to missing IDs, and cycles (with toast notifications explaining what was dropped and why, added *12 June*).

**Benefit:** Matrix organisations can finally be represented — the solid tree stays clean while secondary reporting lines are visible on demand.
**Use cases:** Project leads with functional and delivery managers; chapter/squad models; documenting interim supervision arrangements.

---

## 5. Focusing, filtering and tagging

### Subtree focus ("Show from") *(3 May)*
The chart can be focused on any subtree: pick a role from a hierarchical tree picker (searchable, leading with the incumbent's name), click a node on the chart, and follow the breadcrumb trail back up. The picker layout is tuned to fit the narrow sidebar.

### Tags and dual filters *(1 May)*
Roles gain a free-form tags field, editable in the table and rendered as chips on chart cards. The sidebar offers two ANDed field-filter clauses plus a dedicated Tags row that matches any selected tag. (Tag tokenisation was refined *11 June* to split on whitespace or commas and match whole words.)

**Benefit:** Cuts a large org down to the slice you care about, by structure (subtree), by attribute (filters), or by ad-hoc grouping (tags) — and the three compose.
**Use cases:** Reviewing one division at a time; tagging roles "2027-plan" or "relocating" and viewing just those; filtering to open roles within a focused subtree.

---

## 6. Navigating and viewing the chart

### Zoom controls *(3 May)*
dabeng's built-in wheel zoom — which produced runaway zoom on trackpads — was replaced with a fixed-step, cursor-centred handler, plus sidebar Zoom controls (−, a clickable 100% readout, +).

### Fit-to-view and hotkeys *(3 June)*
A fourth View button scales and recentres the chart so the whole visible structure fits the canvas. Chart-tab-scoped hotkeys cover all zoom actions: `+` / `-` to zoom, `0` to reset, `1` to fit-to-view, each surfaced in its button's tooltip.

### Find box *(12 June)*
A search box on the Chart tab matches against role ID, job title, incumbent name, and department. Repeated searches cycle through matches; for each hit the chart expands any collapsed ancestors and pans the node into view.

**Benefit:** Smooth, predictable movement around charts of any size — and locating a specific person never requires manually expanding branches.
**Use cases:** Presenting (fit the whole org, then dive into a branch); "where does Priya sit?" answered in two keystrokes; cycling between several people with the same surname.

---

## 7. Comparing organisational snapshots *(12 June)*

A Compare… button in the header diffs the currently loaded data against a second CSV/XLSX file. The result classifies every role as added, removed, moved (manager changed), or changed (field edits), and reports headcount and cost deltas. The full diff can be exported as CSV.

**Benefit:** Turns the tool from a viewer/editor into a change-analysis tool — you can quantify exactly what differs between two versions of the org.
**Use cases:** Month-over-month org review against the latest HR extract; validating that a planned reorg file contains only the intended moves; an auditable change list with cost impact for leadership sign-off.

---

## 8. Exports

### Colour key included, in colour *(3 May)*
PNG export snapshots an off-screen clone of the chart plus a horizontal legend strip, so the colour key travels with the image; the print stylesheet lays the legend out as a footer row below the chart. Printing also forces `print-color-adjust: exact` on nodes and legend, so colours appear even when the user hasn't ticked "Background graphics" in the print dialog.

### Filter-aware CSV/Excel export *(11 June)*
Export CSV/Excel exports **what the active tab is showing**, not the raw dataset: grid header filters on the Table tab; filter clauses, tags, and the "Show from" subtree on the Chart and Pivot tabs. Where a role's manager is filtered out, the manager link is rewritten to the nearest visible ancestor so the exported file is still a coherent hierarchy.

### SVG export *(12 June)*
The chart can be exported as SVG (rendered via foreignObject with the styling embedded), alongside PNG and Print.

### Pivot export matches the pivot *(12 June)*
The Pivot tab exports the rendered cross-tab (the aggregated table you see) instead of the underlying raw rows.

**Benefit:** What you see is what you share — exports are self-explanatory (legend included), scoped to the visible slice, and available in a vector format that stays crisp at any size.
**Use cases:** Sending a department head just their own subtree without leaking the wider org's salary data; printed charts readers can interpret without the app; embedding crisp SVG in slide decks; headcount cross-tabs pasted straight into monthly reports.

---

## 9. Colour key (legend) *(4 June)*

The on-screen colour key was reworked:

- **Horizontal bottom strip** — the legend lays out horizontally along the bottom of the chart as a floating overlay, instead of a vertical box in the top-right, reclaiming canvas space.
- **Draggable** — it can be dragged anywhere over the chart, and snaps back to its default lower-left corner whenever the Colour dimension changes.
- **Distinct colours** — the cycling 10-colour palette was replaced with a curated 12-colour set plus a golden-angle fallback generator, so two categories never share a colour no matter how many values the chosen dimension has.
- **Matching exports** *(5 June)* — Print and PNG output place the colour key as a centred horizontal strip at the bottom, matching the on-screen presentation.

**Benefit:** The legend no longer obscures the chart and never produces ambiguous duplicate colours.
**Use cases:** Colouring by a high-cardinality dimension (many departments) without collisions; dragging the key out of the way mid-presentation.

---

## 10. Interface density *(5 May)*

The title bar and file toolbar were merged into a single row (title and privacy info left, filename centre, Open/Save/Save As right), and Load-test-data and Export CSV/Excel moved to the Table tab beside Add Row. The chart/pivot sidebar's narrow edge tab was replaced with a Hide-controls button and per-section accordions (View / Data / Export; Dimensions / Value / Filter), all persisted to localStorage. The header's privacy info icon became a real button opening a popover (dismissable by clicking outside, Escape, or clicking again) instead of a hover-only native tooltip.

**Benefit:** More vertical space for the chart itself, controls collapse to exactly the sections you use, and the layout is remembered between visits.
**Use cases:** Working on a laptop screen; hiding all controls for a clean view while presenting.

---

## Defects fixed

- **Grid edits silently broken under Tabulator v6** *(1 May)* — the table's top-level `cellEdited` callback option was ignored, so cell edits never reached the data layer; validation, Role ID cascade to children, manager-department pre-fill, and chart re-render after edits were all restored by migrating to a live event listener.
- **Grid edits bypassed undo** *(11 June)* — cell edits never pushed undo snapshots (and Role ID renames never propagated) because Tabulator mutates row data before `cellEdited` fires. Fixed; grid edits are now properly undoable.
- **Manual collapse lost on re-render** *(5–6 June)* — manually collapsed branches are preserved across drag-and-drop edits, layout and colour changes (collapsed role IDs are tracked and reapplied after each render), and collapses driven by the node's bottom-edge control — previously never recorded in the default top-down layout — are now tracked too, including the dotted-line overlay redraw. Collapses are intentionally cleared when Depth, filters, or the "Show from" focus change.
- **Wide-vertical layout connectors** *(8 May)* — left-column cards in the 2-column layout were not connected to the centre spine.
- **Legend hidden or duplicated** *(5 June)* — a duplicate legend element was removed and the legend lifted above the fixed Summary panel, so it is no longer hidden behind the Summary view or sidebar after a re-draw.
- **Connectors lingering below collapsed subtrees** *(1 May)* — dabeng's buggy `visibleLevel` was replaced with manual collapse classes in hybrid mode; also fixed the Cost field gate so a zero value renders consistently with cumulative fields.
- **Sidebar dropdowns clipped** *(3 May)* — dropdowns are attached to `<body>` so they escape the sidebar's overflow clipping.
- **Dialogs opened in the top-left corner** *(11 June)* — the global CSS margin reset was wiping the browser's native dialog centring; dialogs now open centred.
- **Chart click handler self-cancelling** *(6 May)* — the old reparent panel closed itself on the same event bubble that opened it, making the chart effectively read-only; the dead panel was removed as part of chart-side editing.

## Other minor items

- Tag filters split tags on whitespace **or** commas and match whole words *(11 June)*.
- Invalid dotted-line manager IDs found during import are scrubbed with a toast notification explaining which IDs were dropped and why *(12 June)*.
- The Layout selector was dropped *(3 May)* and the remaining dead `top`/`left` presets removed *(12 June)*; the top-down hybrid is the single supported layout.
- The "Show from" tree picker was tightened to fit the narrow sidebar and lead with the incumbent's name *(3 May)*.
