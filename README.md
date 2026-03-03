# Browser Utilities

A collection of small, standalone, **ad-free** browser tools. No accounts, no tracking, no nonsense.

Live at: **[https://www.duplessis.uk](https://www.duplessis.uk)**

---

## Tools

### 🌍 Timezone Compare
Compare times across multiple timezones at a glance. Great for scheduling international meetings and calls. Supports shareable links so you can send the same view to colleagues.

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

---

## Design philosophy

- **Zero dependencies** — each tool is a single self-contained `.html` file with inline CSS and JavaScript.
- **No build step** — open any file directly in a browser, or serve the folder with any static file server.
- **No back-end** — all computation runs in the browser; the only network calls are to the Frankfurter exchange-rate API and CDN resources (Chart.js).
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
