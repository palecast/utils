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

Most tools share the same purple gradient theme (`#667eea → #764ba2`). 

**Standard tool structure:**
- System font stack: `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, ...`
- Responsive via media queries at `max-width: 768px`
- White card(s) on gradient background

### External Dependencies (CDN)

- **Chart.js** (`cdn.jsdelivr.net/npm/chart.js`) — used by `currency-converter.html` and `ccy-tracker.html`
- **Google Fonts** — used by `ccy-tracker.html` (DM Serif Display, Work Sans)
- **frankfurter.dev API** (`api.frankfurter.dev/v1`) — live and historical ECB exchange rates, used by both currency tools. No API key required.

### State Persistence

`ccy-tracker.html` saves user state (selected currencies, time period, base currency) to `localStorage` under the key `currencyTrackerState`.

### No Back-End

All computation happens in the browser. The only network calls are to the frankfurter.dev API (currency tools) and CDN resources.
