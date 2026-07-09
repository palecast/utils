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
- `org-chart-guide.html` — Static how-to guide for the Org Chart tool, linked from its header Help button
- `dc-inventory.html` — **Obsolete**: unmaintained, deliberately absent from `index.html` (role-gated in `staticwebapp.config.json`); do not extend

### Visual Design Conventions

See [DESIGN.md](DESIGN.md) for the full style guide (colors, typography, components, motion, responsive rules). Quick summary:

- **Palette**: warm off-white page (`#FAF8F5`), sage-green accent (`#4A5D47`), warm brown text (`#3A2F28` / `#6B5F54`). Shadows and borders are brown/accent-tinted — never black or gray.
- **Font**: Inter (weights 300/400/500/600) from Google Fonts, with a system-font fallback stack.
- **Layout**: white cards on the off-white background (not on a gradient); `border-radius` ladder of 8 / 12 / 16; single responsive breakpoint at `max-width: 768px`.
- **One-offs**: `ccy-tracker.html` uses `DM Serif Display` for its distinct branding — intentional exception, not the standard.

When adding or modifying a tool, follow DESIGN.md rather than copying styles from an arbitrary existing file — older tools may still reflect an earlier purple-gradient theme.

### External Dependencies (CDN)

- **Inter** (`fonts.googleapis.com/css2?family=Inter`) — loaded by every standard tool (weights 300/400/500/600)
- **Chart.js** (`cdn.jsdelivr.net/npm/chart.js`) — used by `currency-converter.html`, `ccy-tracker.html`, and `loan-calculator.html`
- **qrcode-generator** (`cdnjs.cloudflare.com/ajax/libs/qrcode-generator/1.4.4`, SRI-pinned) — used by `qr-generator.html` (UTF-8-correct QR encoding)
- **DM Serif Display, Work Sans** (Google Fonts) — used only by `ccy-tracker.html` (intentional design exception)
- **Tabulator** (`cdn.jsdelivr.net/npm/tabulator-tables@6`) — used by `org-chart.html` for the table/grid view
- **OrgChart (Dabeng)** (`cdn.jsdelivr.net/npm/orgchart@5`) + **jQuery 3.7.1** — used by `org-chart.html` for the tree visualization
- **html2canvas** (`cdn.jsdelivr.net/npm/html2canvas@1.4.1`) — used by `org-chart.html` for PNG export/print
- **SheetJS (xlsx)** (`cdn.jsdelivr.net/npm/xlsx`) — used by `org-chart.html` for Excel import/export
- **pdf-lib** (`cdn.jsdelivr.net/npm/pdf-lib@1.17.1`, SRI-pinned) — used by `pdf-toolbox.html` for PDF assembly (merge/split/rotate/extract)
- **pdfjs-dist** (`cdn.jsdelivr.net/npm/pdfjs-dist@6.1.200`, ESM dynamic import + CDN worker via `GlobalWorkerOptions.workerSrc`) — used by `pdf-toolbox.html` for page thumbnails. v4+ is ESM-only, so no SRI is possible on the dynamic import; the exact version pin is the mitigation
- **fflate** (`cdn.jsdelivr.net/npm/fflate@0.8.3`, SRI-pinned) — used by `pdf-toolbox.html` for the split-to-ZIP export (store-only zipping)
- **frankfurter.dev API** (`api.frankfurter.dev/v1`) — live and historical ECB exchange rates, used by both currency tools. No API key required.

### State Persistence

Four tools persist user state to `localStorage`:

| Tool | Key | What is saved |
|------|-----|---------------|
| `ccy-tracker.html` | `currencyTrackerState` | Selected currencies, time period, base currency |
| `currency-converter.html` | `currencyConverterState` | From/to currencies, amount, selected period |
| `timezone-compare-app.html` | `world-timeboard-state-v1` | City rows, home timezone, start hour |
| `loan-calculator.html` | `loanCalculatorState` | Loan inputs (amount, rate, term, start month, extra payment, currency symbol) |
| `org-chart.html` | `orgChartUiState` | UI chrome only (sidebar/section collapse state) |
| `org-chart.html` | `orgChartAutosave:<tabId>` | Per-tab crash-recovery snapshot of unsaved working data (debounced; cleared on save/open; restore offered on next visit for snapshots whose tab is dead, judged via the `orgChartAutosaveHb:<tabId>` heartbeat key; tab id lives in `sessionStorage` as `orgChartTabId`) |

`org-chart.html` also uses a `BroadcastChannel` (`org-chart-tabs`) to warn when the same file appears to be open in more than one tab.

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