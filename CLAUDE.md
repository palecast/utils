# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A collection of standalone, ad-free browser utilities. Each tool is a **single self-contained HTML file** with no build system, no package manager, and no server required. Open any `.html` file directly in a browser to run it.

## Running / Development

No build step. Just open files in a browser. For local development with live reload, any simple static file server works:

```bash
python3 -m http.server 8000
# or
npx serve .
```

## Architecture

### Adding a New Tool

1. Create a new `tool-name.html` file (self-contained: HTML + inline CSS + inline `<script>`)
2. Register it in `index.html` by adding an entry to the `utilities` array in the `<script>` block

### File Structure

- `index.html` — Landing page; dynamically renders tool cards from a `utilities[]` config array. Page title changes based on `window.location.hostname`.
- Each `*.html` — A standalone utility with no external file dependencies (except CDN links)
- `disclaimer.html` — Shared terms/disclaimer page linked from tool footers

### Visual Design Conventions

See [DESIGN.md](DESIGN.md) for the full style guide (colors, typography, components, motion, responsive rules). Quick summary:

- **Palette**: warm off-white page (`#FAF8F5`), sage-green accent (`#4A5D47`), warm brown text (`#3A2F28` / `#6B5F54`). Shadows and borders are brown/accent-tinted — never black or gray.
- **Font**: Inter (weights 300/400/500/600) from Google Fonts, with a system-font fallback stack.
- **Layout**: white cards on the off-white background (not on a gradient); `border-radius` ladder of 8 / 12 / 16; single responsive breakpoint at `max-width: 768px`.
- **One-offs**: `ccy-tracker.html` uses `DM Serif Display` for its distinct branding — intentional exception, not the standard.

When adding or modifying a tool, follow DESIGN.md rather than copying styles from an arbitrary existing file — older tools may still reflect an earlier purple-gradient theme.

### External Dependencies (CDN)

- **Inter** (`fonts.googleapis.com/css2?family=Inter`) — loaded by every standard tool (weights 300/400/500/600)
- **Chart.js** (`cdn.jsdelivr.net/npm/chart.js`) — used by `currency-converter.html` and `ccy-tracker.html`
- **DM Serif Display, Work Sans** (Google Fonts) — used only by `ccy-tracker.html` (intentional design exception)
- **Tabulator** (`cdn.jsdelivr.net/npm/tabulator-tables@6`) — used by `org-chart.html` for the table/grid view
- **d3-org-chart** (`cdn.jsdelivr.net/npm/d3-org-chart@3`) + **d3-flextree** — used by `org-chart.html` for the tree visualization
- **SheetJS (xlsx)** (`cdn.jsdelivr.net/npm/xlsx`) — used by `org-chart.html` for Excel import/export
- **frankfurter.dev API** (`api.frankfurter.dev/v1`) — live and historical ECB exchange rates, used by both currency tools. No API key required.

### State Persistence

Three tools persist user state to `localStorage`:

| Tool | Key | What is saved |
|------|-----|---------------|
| `ccy-tracker.html` | `currencyTrackerState` | Selected currencies, time period, base currency |
| `currency-converter.html` | `currencyConverterState` | From/to currencies, amount, selected period |
| `timezone-compare-app.html` | `world-timeboard-state-v1` | City rows, home timezone, start hour |

`cost-splitter.html` and `days-between.html` do **not** persist state.

### Security Conventions

When user-supplied strings are inserted into HTML via template literals or `innerHTML`, always escape them first. `cost-splitter.html` has a reusable `escapeHtml()` function as a reference:

```js
function escapeHtml(str) {
    return String(str)
        .replace(/&/g, '&amp;')
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt;')
        .replace(/"/g, '&quot;');
}
```

Similarly, `qr-generator.html` has `escapeVCardField()` (escapes `\`, `;`, `,`) and `escapeWiFiString()` (escapes `\`, `;`, `,`, `:`, `"`) for format-specific output.

### No Back-End

All computation happens in the browser. The only network calls are to the frankfurter.dev API (currency tools) and CDN resources.

### Hosting Infrastructure

Deployed as an **Azure Static Web App** (Free tier), auto-deployed from the `main` branch via GitHub Actions.

| Property | Value |
|----------|-------|
| Azure resource name | `Utils` |
| Resource group | `rg-static-web` |
| Region | Central US |
| Default hostname | `victorious-island-015b3f010.3.azurestaticapps.net` |
| GitHub repo | `https://github.com/palecast/utils` |
| Deploy branch | `main` |

Pushing to `main` triggers deployment automatically — no manual deploy step needed.

To inspect or manage via Azure CLI:
```bash
az staticwebapp show --name Utils --resource-group rg-static-web
az staticwebapp list --output table
```