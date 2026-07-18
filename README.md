# Browser Utilities

A collection of small, standalone, **ad-free** browser tools. No accounts, no tracking, no nonsense.

Live at: **[https://www.duplessis.uk](https://www.duplessis.uk)**

---

## Tools

### 🌍 Timezone Compare
Compare times across different timezones easily. Perfect for scheduling international meetings and calls. Supports shareable links so you can send the same view to colleagues.

### 📅 Days Between
Calculate the number of days between two dates, including working days and public holidays.

### 🔲 QR Code Generator
Generate QR codes for URLs, email addresses, Wi-Fi networks, and vCards. All generated client-side — nothing is uploaded.

### 💱 Currency Converter
Convert between currencies and view a chart of historical exchange rates. Powered by the [Frankfurter API](https://www.frankfurter.dev) (ECB rates, no API key required).

### 💸 Currency Value Tracker
Compare the relative value of currencies over time against a chosen base currency. Useful for spotting long-term trends. Also uses Frankfurter/ECB data.

### 💰 Cost Splitter
Split a bill between any number of people, with support for individual non-shared items and optional tip.

### 🏢 Org Chart
Design and model org structures. Import/export Excel/CSV, filter by department or role, color-code nodes, and visualize hierarchies as a tree or table. Compare snapshots, merge delegated edits back in, and export to print, PNG, SVG, or PowerPoint SmartArt. Has its own [how-to guide](https://www.duplessis.uk/org-chart-guide).

### 🗄️ DC Room Viewer
Design and visualise data-centre room layouts to scale: place racks, PDUs and cooling from a component library onto a tile grid, draw walls, doors, cages and containment, colour equipment by customer or power draw, and export to PNG or PDF. Rooms and component libraries are plain JSON files. Has its own [how-to guide](https://www.duplessis.uk/dc-room-viewer-guide).

### ☀️ Daylight Hours
Visualize sunrise and sunset for every day of the year for any city, as a ribbon chart with hover details.

### 🌊 Coastal Conditions
Marine conditions for watersports: wave height, swell, sea temperature, and currents at coastal locations worldwide. Powered by Open-Meteo.

### 🔄 Field Mapper
Map and transform CSV/Excel fields with template expressions — drag source fields into output expressions, preview live, export CSV/Excel.

### 📝 Text Diff
Compare two text files or snippets side by side, with added/removed/changed highlighting and change navigation.

### 📄 Markdown Editor
Write Markdown with a live side-by-side preview. Open and save `.md` files locally; export to PDF with selectable text.

### 🧰 JSON Formatter
Format, minify and validate JSON. Explore documents in a collapsible tree with search and path copying.

### 🏦 Loan Calculator
Amortization schedules with monthly payment, total interest, payoff date, and the effect of extra payments. Export the schedule to CSV.

---

## Design philosophy

- **Zero dependencies** — each tool is a single self-contained `.html` file with inline CSS and JavaScript.
- **No build step** — open any file directly in a browser, or serve the folder with any static file server.
- **No back-end** — all computation runs in the browser; the only network calls are to the Frankfurter exchange-rate API and CDN resources.
- **Privacy-first** — no analytics, no ads, no external tracking.

---

## Running locally

```bash
# Python
python3 -m http.server 8000

# Node
npx serve .
```

Then open `http://localhost:8000` in your browser.

---

## Adding a new tool

1. Create a new `tool-name.html` (self-contained: HTML + inline CSS + inline `<script>`).
2. Add an entry to the `utilities[]` array in `index.html`.
