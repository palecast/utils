# DESIGN.md — UI Style Guide

A portable design spec for single-file browser utilities. Drop this into another project, and Claude Code can use it as the source of truth when updating that project's UI to match.

The aesthetic: **warm, neutral, editorial**. Sage-green accent on an off-white page. Quiet shadows. Generous whitespace. Inter everywhere. One breakpoint. No framework — every rule below works as plain CSS.

---

## 1. Color palette

### Core neutrals & accent

| Token | Hex | Usage |
|-------|-----|-------|
| `--bg` | `#FAF8F5` | Page background (warm off-white) |
| `--surface` | `#FFFFFF` | Cards, panels, inputs |
| `--text` | `#3A2F28` | Primary text, headings |
| `--text-muted` | `#6B5F54` | Secondary text, labels, captions |
| `--accent` | `#4A5D47` | Primary brand (sage green) — buttons, active states, logo, section headings |
| `--accent-hover` | `#3D4E3B` | Primary button hover |
| `--accent-soft` | `#8B6F47` | Warm tan — used as gradient endpoint and secondary accent |
| `--border` | `rgba(74, 93, 71, 0.12)` | Default card/input border (accent-tinted, very soft) |
| `--border-strong` | `rgba(74, 93, 71, 0.2)` | Emphasized borders (tinted cards) |
| `--tint` | `rgba(232, 229, 223, 0.4)` | Info panels, subtle surface fills |
| `--pill-bg` | `#E8E5DF` | Inactive tabs, favorite pills |

### Gradients

```css
/* Primary CTA gradient */
background: linear-gradient(135deg, #4A5D47 0%, #8B6F47 100%);

/* Success / positive actions */
background: linear-gradient(135deg, #43a047 0%, #66bb6a 100%);
```

Use the primary gradient sparingly — on one or two hero buttons per page. Most buttons should be solid `--accent`.

### Semantic colors

| State | Text | Background | Border |
|-------|------|------------|--------|
| Success | `#2980b9` | `#d4edda` | — |
| Error | `#c33` | `#fef2f2` | `#fecaca` (+ `4px solid #c33` left) |
| Warning | `#856404` | `#fff3cd` | `#ffc107` (+ `4px solid #ffc107` left) |
| Info (neutral) | `#6B5F54` | `rgba(232, 229, 223, 0.4)` | `rgba(74, 93, 71, 0.08)` |

Destructive button: `#e74c3c` → hover `#c0392b`.
Secondary (cancel) button: `#95a5a6` → hover `#7f8c8d`.

### What this palette is NOT

Do not use saturated blues, purples, or bright greens as the primary brand color. If an older project uses `#667eea → #764ba2` (purple gradient) or similar, replace it with the sage palette above. Keep bright colors only for semantic states (success/error/warning) or domain-specific data visualization (e.g., temperature ramps).

---

## 2. Typography

### Font stack

```css
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600&display=swap');

font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI',
             Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
```

Inter is the only font. Load weights **300, 400, 500, 600**. Do not pull in 700 or heavier — emphasis should come from 600, not black. (A display serif like DM Serif Display is acceptable for one tool's branding but is not part of the system.)

### Scale

| Role | Size | Weight | Color | Notes |
|------|------|--------|-------|-------|
| Hero h1 | `2–2.5rem` (desktop) / `1.6rem` (mobile) | 600 | `--text` | Page titles |
| Section h2 | `1.25–1.5rem` | 500–600 | `--accent` | Section dividers |
| Card h3 | `1rem–1.05rem` | 500–600 | `--text` or `--accent` | Card titles |
| Body | `14px–1rem` | 400 | `--text` | Line-height `1.5–1.6` |
| Lead / hero description | `1.1–1.125rem` | 300 | `--text-muted` | Line-height `1.6–1.7` |
| Label / eyebrow | `10–12px` | 600 | `--text-muted` | `text-transform: uppercase`, `letter-spacing: 0.9–1.2px` |
| Caption / meta | `0.75–0.875rem` | 400 | `--text-muted` | |

Heading line-height: `1.1–1.2`. Body: `1.5–1.7`. Compact UI rows: `1–1.2`.

---

## 3. Layout

### Container

- Standard tool: `max-width: 900–1000px`, centered, `padding: 20px` (12px on mobile).
- Dashboard/hub: `max-width: 1200px`.
- Wide data tools (org charts, boards): `max-width: 1400px`.

### Grid

- Card hub grids: `repeat(4, 1fr)` → `repeat(3, 1fr)` at ~1024px → `repeat(2, 1fr)` at 768px → `1fr` at 480px.
- Two-panel layouts: `1fr 1fr` → `1fr` at 768px.
- Metric rows: `repeat(3–4, 1fr)` → `1fr` at 768px.
- Default gap: `14–20px`.

### Spacing scale (approximate — do not be religious about it)

| Step | px |
|------|----|
| Tiny | 4–6 |
| Small | 8–12 |
| Regular | 16–20 |
| Large | 24–30 |
| XL | 32–48 |

---

## 4. Cards & surfaces

```css
.card {
  background: #FFFFFF;
  border: 1px solid rgba(74, 93, 71, 0.12);
  border-radius: 12px;
  padding: 24px;
  box-shadow: 0 10px 30px rgba(58, 47, 40, 0.08);
}
```

**Border-radius ladder:**
- `8px` — inputs, small buttons
- `12px` — standard cards, logo box
- `14px` — metric/data cards
- `16px` — large info panels, result cards
- `20px` — hero/showcase containers (rare)
- `20–25px` (pill) — tabs, period selectors, favorite pills
- `50%` — round icon buttons (swap, favorite)

**Shadow ladder** (all use `rgba(58, 47, 40, X)` — brown-tinted, never black):
- Subtle: `0 2px 8px rgba(58, 47, 40, 0.04)` — metric cards
- Standard: `0 10px 30px rgba(58, 47, 40, 0.08)` — cards
- Lifted: `0 20px 60px rgba(58, 47, 40, 0.12)` — hero cards, modals

**Tinted card variant** (info panels): background `rgba(232, 229, 223, 0.4)`, border `rgba(74, 93, 71, 0.06)`, `border-radius: 16px`, padding up to `48px`.

---

## 5. Components

### Buttons

```css
.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  background: #4A5D47;
  color: #FFFFFF;
  font: 600 14px 'Inter', sans-serif;
  cursor: pointer;
  transition: background 0.2s, transform 0.1s;
}
.btn:hover { background: #3D4E3B; transform: translateY(-1px); }
.btn:active { transform: translateY(0); }
```

Variants:
- **Secondary** — background `#95a5a6`, hover `#7f8c8d`.
- **Outline** — background `#FFFFFF`, border `1px solid rgba(74, 93, 71, 0.12)`, text `#4A5D47`. No `translateY` on hover.
- **Danger** — background `#e74c3c`, hover `#c0392b`.
- **Small** — `padding: 5px 10px; font-size: 12px`.
- **Round icon** — `50px × 50px`, `border-radius: 50%`.

### Inputs & selects

```css
input, select {
  padding: 10px 12px;
  border: 2px solid rgba(74, 93, 71, 0.12);
  border-radius: 8px;
  font: 400 14px 'Inter', sans-serif;
  background: #FFFFFF;
  transition: border-color 0.3s;
}
input:focus, select:focus {
  outline: none;
  border-color: #4A5D47;
}
```

Keep the 2px border — the focus state depends on the color change being visible.

### Tabs / segmented controls

Pill style, 20px radius:

```css
.tab {
  padding: 8px 16px;
  border: 2px solid transparent;
  border-radius: 20px;
  background: #E8E5DF;
  font: 600 13px 'Inter', sans-serif;
  cursor: pointer;
  transition: all 0.2s;
}
.tab.active {
  background: #FFFFFF;
  border-color: #4A5D47;
}
```

Period-selector variant uses `border-radius: 25px` with an accent border on both active and inactive states.

### Badges / pills

- **Info badge**: `8px 16px`, `border-radius: 10px`, white bg, soft border.
- **Favorite/tag pill**: `5–6px 10–12px`, `border-radius: 20px`, `#E8E5DF` bg, `#4A5D47` text, `0.8–0.85rem`.

### Status banners

Left-border accent pattern:

```css
.error  { background:#fef2f2; border:1px solid #fecaca; border-left:4px solid #c33;    color:#c33; }
.warn   { background:#fff3cd; border:1px solid #ffc107; border-left:4px solid #ffc107; color:#856404; }
.success{ background:#d4edda;                                                           color:#2980b9; }
```

All use `padding: 15px 22px; border-radius: 8px`.

### Loading indicator

A 3px shimmer bar, not a spinner:

```css
.loading-bar {
  height: 3px;
  background: rgba(74, 93, 71, 0.12);
  overflow: hidden;
  position: relative;
}
.loading-bar::after {
  content: '';
  position: absolute; inset: 0;
  background: linear-gradient(90deg, transparent, #4A5D47, transparent);
  animation: loadSlide 1.4s ease-in-out infinite;
}
@keyframes loadSlide {
  from { transform: translateX(-150%); }
  to   { transform: translateX(350%); }
}
```

---

## 6. Header & footer

### Sticky header

```css
header {
  position: sticky; top: 0; z-index: 50;
  padding: 16px 24px;
  background: rgba(255, 255, 255, 0.5);
  backdrop-filter: blur(12px);
  border-bottom: 1px solid rgba(74, 93, 71, 0.12);
  display: flex; justify-content: space-between; align-items: center;
}
```

**Logo box**: `40px × 40px`, `border-radius: 12px`, background `#4A5D47`, color `#FAF8F5`, centered initial in `600 14px`.
**Title**: `600 16px`, color `#3A2F28`, `letter-spacing: -0.01em`.
**Subtitle**: `12px`, color `#6B5F54`.

### Footer

```css
footer {
  padding: 32px 24px;
  text-align: center;
  font: 300 0.875rem 'Inter', sans-serif;
  color: #6B5F54;
  border-top: 1px solid rgba(74, 93, 71, 0.12);
  background: rgba(255, 255, 255, 0.3);
}
footer a {
  color: #6B5F54;
  text-decoration: none;
  transition: color 0.2s;
}
footer a:hover { color: #3A2F28; }
```

Multiple footer items: flex with `gap: 24px`.

---

## 7. Motion

One primary breakpoint; one primary motion vocabulary.

| Interaction | Transform | Duration |
|-------------|-----------|----------|
| Button hover | `translateY(-1px)` | `0.1–0.2s` |
| Card hover (clickable) | `translateY(-5px)` | `0.3s ease` |
| Color/border transitions | — | `0.2–0.3s` |
| Loading shimmer | `translateX(-150% → 350%)` | `1.4s ease-in-out infinite` |

Default timing function is `ease`. Avoid bounces, long durations, or `cubic-bezier` overrides unless there's a specific reason.

---

## 8. Responsive

One breakpoint does most of the work:

```css
@media (max-width: 768px) {
  /* Grids collapse to 1–2 columns */
  /* h1 shrinks (~2.5rem → 1.6rem) */
  /* Side-by-side panels stack */
  /* Card padding: 24–30px → 18–20px */
  /* Header nav hides */
}
```

Optional secondary steps:
- `max-width: 1024px` — 4-col grid → 3-col.
- `max-width: 480px` — body padding `20px → 12px`, card grid → 1 col.

Avoid proliferating breakpoints. If something needs a third breakpoint, question the layout first.

---

## 9. Security conventions (carry these over too)

When updating UI code, preserve these patterns — they are part of how these tools are built:

- Always escape user-supplied strings before injecting into HTML via template literals or `innerHTML`. Reuse the `escapeHtml()` helper pattern:
  ```js
  function escapeHtml(str) {
    return String(str)
      .replace(/&/g, '&amp;').replace(/</g, '&lt;')
      .replace(/>/g, '&gt;').replace(/"/g, '&quot;');
  }
  ```
- For format-specific output (vCard, Wi-Fi QR, CSV), write a format-specific escape function rather than reusing HTML escaping.
- No inline event handlers in templates (`onclick="..."` with interpolated values). Attach via `addEventListener` after render.

---

## 10. Quick checklist for "does this match the system?"

- [ ] Page background is `#FAF8F5`, not pure white.
- [ ] Accent color is sage `#4A5D47`, not purple or blue.
- [ ] Text is `#3A2F28` (headings/body) and `#6B5F54` (muted) — no pure black.
- [ ] Inter is loaded from Google Fonts with weights 300/400/500/600.
- [ ] Shadows are brown-tinted (`rgba(58, 47, 40, …)`), not black.
- [ ] Borders are accent-tinted (`rgba(74, 93, 71, 0.12)`), not gray.
- [ ] Border-radius follows the 8 / 12 / 16 ladder.
- [ ] Buttons lift `1px` on hover; cards lift `5px` if clickable.
- [ ] Labels/eyebrows are uppercase, 10–12px, letter-spaced.
- [ ] One `@media (max-width: 768px)` block, not five.
- [ ] No emojis in UI unless explicitly part of the content.
