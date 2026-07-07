# App Review — July 2026

A full code review of every utility in the repo (~20k lines across 14 tools), covering latent defects, enhancement opportunities, and new-app candidates. High-priority defects were **fixed in the same pass** as this review (7 July 2026) and are marked ✅ below; everything else is a documented backlog.

**Effort scale:** **XS** < 15 min · **S** 15–60 min · **M** 1–3 h · **L** a day or more.

---

## Cross-cutting findings

1. **UTC/DST date handling is the most common bug class.** Five apps (ccy-tracker, currency-converter, days-between, daylight-hours, coastal-conditions) mix local-time `Date` math with UTC ISO strings, producing off-by-one-day or off-by-one-hour errors near midnight, at negative UTC offsets, or across DST transitions. A shared "UTC date helpers" snippet applied consistently would eliminate the class. [S per app]
2. **Fetch hygiene was inconsistent.** coastal-conditions was the model (AbortController, `response.ok`, graceful degradation); the currency tools had none of it. ✅ Both currency tools now abort stale requests, debounce, check `response.ok`, and show inline errors. days-between now uses `Promise.allSettled` with a capped year range.
3. **XSS discipline is now consistent.** Most tools escape correctly or build DOM via `textContent`; the two unescaped API-string sinks in days-between are fixed ✅, and org-chart's one inconsistent sink (pivot data cells) is defensively escaped ✅.
4. **Theme migration is complete.** No tool uses the old purple gradient. Remaining deviations are polish-level (listed per app).
5. **CDN includes carry no Subresource Integrity.** The new qr-generator include is SRI-pinned ✅; the rest (Chart.js, Tabulator, SheetJS, jQuery, orgchart, html2canvas, marked/DOMPurify/hljs, diff) are unpinned and unhashed. Pinning + SRI site-wide is a single coherent change. [M]
6. **Route registration was easy to forget.** Four tools were linked from the index but had no rewrite rule in `staticwebapp.config.json`, so their cards 404'd on the deployed site. Fixed ✅ — and worth automating with a manifest-driven check. [S]

---

## Per-app review

### index.html (hub)
| Defect | Severity | Status |
|---|---|---|
| Cards for daylight-hours, field-mapper, text-diff, markdown-editor 404'd on Azure (no rewrite rules) | High | ✅ Fixed — routes added |
| dc-inventory absent from hub (commented out) | Info | ✅ Marked obsolete deliberately |

**Enhancements:** manifest-driven card/route generation so a new tool can't miss registration [S]; `<noscript>` fallback list [XS].

### org-chart.html
| Defect | Severity | Status |
|---|---|---|
| `recomputeNextId` throws on null `roleId` — aborts autosave restore / merge apply mid-way | High | ✅ Fixed |
| Roles in a detached reporting cycle silently vanish from chart/pivot/summary/exports | High | ✅ Fixed — promoted to top level + named in banner |
| Grid Role-ID edit colliding with an existing ID silently merged two roles' reports | High | ✅ Fixed — collision rejected with toast (form editor parity) |
| Merge wrote Department via manager-inheritance even when the file had no Department column | Medium | ✅ Fixed — inheritance suppressed unless column present |
| Depth control off-by-one with synthetic root (first “−” presses did nothing) | Medium | ✅ Fixed |
| Pivot data cells unescaped innerHTML (numeric today, fragile tomorrow) | Low | ✅ Fixed defensively |
| Duplicate Role IDs collapse last-wins in Compare/Merge/export maps → under-reported diffs, wrong-instance merges | Medium | Deferred — proper fix is duplicate-ID handling on import (block or auto-suffix) [M] |
| Autosave restore / undo don't restore filter selections — restored roles can be invisibly filtered | Low | Deferred [M] |
| Context-menu outside-click listener not removed on item-click close path | Low | Deferred [XS] |
| Legend drag adds permanent document-level listeners | Low | Deferred [XS] |
| Multi-tab warning matches filename only (false positives across folders) | Low | Deferred — documented in code |
| `pagehide` "closed" broadcast best-effort → multi-tab banner can linger | Low | Deferred [S] |
| Tabulator `getData('active')` vs `replaceData` timing for filtered export | Low | Deferred — needs verification [S] |
| `manuallyCollapsed` set unbounded within a session | Very low | Deferred |

**Enhancements:** JSON save format for lossless round-trips incl. UI state [S/M]; org-health metrics (layers, spans, vacancy aging) [M]; photos/avatars on cards [M/L]; capture filter state in undo snapshots [M]; scenario branching (in-app what-if compare) [L].

### days-between.html
| Defect | Severity | Status |
|---|---|---|
| XSS: holiday names and subdivision names from the nager.at API injected unescaped | High | ✅ Fixed |
| Unbounded `addWorkingDays` loop could hang the tab; uncapped per-year holiday fetches; one failed year dropped all holidays | High | ✅ Fixed — clamps, 20-year cap with notice, `allSettled` |
| DST fall-back day inflates total-days count by 1 (local-time `Math.ceil` diff) | Medium | Deferred — switch to `Date.UTC` diff [S] |
| Total days exclusive-ish vs working days inclusive — Mon→Fri shows 4 vs 5 | Medium | Deferred — clarify labels or align conventions [S] |
| `counties === null` assumption for nationwide holidays | Low | Deferred [XS] |
| Editable result numbers are divs with onclick (no keyboard access) | Low | Deferred [S] |

**Enhancements:** cache fetched holiday years across country switches [S]; date-range presets ("this quarter", "to year-end") [S].

### timezone-compare-app.html
| Defect | Severity | Status |
|---|---|---|
| Invalid timezone in a shared URL (or corrupted localStorage) crashed the whole board | High | ✅ Fixed — zones validated, bad rows dropped |
| DST-day column bug: home offset evaluated at wrong instant → wrong hour on transition days | Medium | Deferred [M] |
| Offset badge and remove control are divs, not buttons (no keyboard/AT access) | Low | Deferred [S] |
| Hover overlay 1px drift (border not included in cell-width math) | Low | Deferred [XS] |

**Enhancements:** copy-as-text table [S]; current-time marker line [S]; visible feedback when clipboard copy fails [XS].

### ccy-tracker.html
| Defect | Severity | Status |
|---|---|---|
| No AbortController → out-of-order responses could show a stale chart | High | ✅ Fixed |
| Redundant second fetch per update just for x-axis labels (worst on 5-year range) | Medium | ✅ Fixed — labels reused from first dataset |
| No `response.ok` checks; failures left selects stuck on "Loading…"; errors via `alert()` | Medium | ✅ Fixed — inline error banner |
| Empty-range NaN (range landing entirely on closed days) | Low | Deferred [S] |
| UTC off-by-one in `getDateRange` at negative offsets late in the day | Low | Deferred [S] |
| Inline `onclick` with interpolated currency code in pills | Low | Deferred (API-controlled values) [XS] |
| Theme polish: h1 weight 700; spinner instead of the shimmer bar | Low | Deferred [XS] |

**Enhancements:** cache currency list for offline render [S]; share fetch layer with currency-converter [M].

### currency-converter.html
| Defect | Severity | Status |
|---|---|---|
| Fetch per keystroke, no debounce/abort → API flood + stale results | High | ✅ Fixed — 250ms debounce + AbortController |
| No `response.ok` checks | Medium | ✅ Fixed |
| Tooltip/date UTC-vs-local off-by-one (previous day shown at negative offsets) | Low | Deferred [S] |
| Theme polish: cool-gray section bg, `h1 #333`, emoji in title | Low | Deferred [XS] |

**Enhancements:** single range request serving both conversion and chart [S]; cache last successful rate for offline conversion [S].

### daylight-hours.html
| Defect | Severity | Status |
|---|---|---|
| Hardcoded UTC offsets ignore DST — displayed clock times off ~1h during DST (disclosed in a notice) | Medium | Deferred — DST toggle or IANA-based offsets [M] |
| Tooltip time can disagree with drawn ribbon at polar extremes (clamp vs mod-1440) | Low | Deferred [S] |
| `getDayOfYear` local-time math can misplace the Today marker on DST days | Low | Deferred [XS] |
| `TODAY_DOY` computed once — stale after midnight | Low | Deferred — recompute on `visibilitychange` [XS] |
| Favourites entries not shape-validated (corrupted entry breaks pills) | Low | Deferred [XS] |
| Canvas has no text alternative (a11y) | Low | Deferred — ARIA live summary [S] |
| Theme: indigo/purple accents inside the chart widget (tooltip, stat pill, gray input border) | Low | Deferred [S] |

**Enhancements:** year selector (engine already takes a year) [S]; city comparison overlay [M].

### coastal-conditions.html
| Defect | Severity | Status |
|---|---|---|
| "Now" hourly highlight and "Updated" time use the viewer's timezone, not the location's | Medium | Deferred [S] |
| "Tides" card is a moon-phase spring/neap proxy with no disclaimer — plausibly misleading | Medium | Deferred — relabel + disclaimer [XS] |
| "7-Day Forecast" heading renders 8 days (`forecast_days=8`) | Low | Deferred [XS] |
| Geocoding fetch lacks `response.ok`; failures silently show no results | Low | Deferred [XS] |

*Otherwise the reference implementation: AbortController, `response.ok`, typed state validation, `textContent`-only rendering, DESIGN.md-exact styling.*

### cost-splitter.html
| Defect | Severity | Status |
|---|---|---|
| `paidBy` stored by name — rename/delete orphaned the reference, silently corrupting settlements | High | ✅ Fixed — renames follow references; deletions reset to shared with a notice |
| Duplicate person names merge into one settlement balance | Medium | Deferred — needs stable person IDs [M] |
| Displayed shares don't reconcile to totals ($10 ÷ 3 → 3×$3.33 = $9.99) | Medium | Deferred — largest-remainder allocation [M] |
| Settlement penny drift (0.005 thresholds + display rounding) | Low | Deferred — same fix as above [—] |
| `parseFloat` accepts `Infinity` for cost amounts | Low | Deferred [XS] |
| `calculateTotals()` re-run redundantly per render and mutates percent-cost amounts | Perf | Deferred [S] |

**Enhancements:** JSON export/import of scenarios [S]; per-scenario currency [S].

### qr-generator.html
| Defect | Severity | Status |
|---|---|---|
| qrcodejs 1.0.0 sized QR versions by JS string length → non-ASCII (accents, emoji, non-Latin SSIDs) corrupted or threw; emoji encoded as CESU-8 | High | ✅ Fixed — replaced with `qrcode-generator@1.4.4` (SRI-pinned, UTF-8 byte encoder), canvas rendered locally |
| vCard: newlines in Address broke the record; TEL/EMAIL/URL unescaped | Medium | ✅ Fixed — `\n` folding + all fields escaped |
| WiFi `H:` field always emitted (`H:false` confuses some scanners) | Low | Deferred [XS] |
| No CDN-failure message (generic alert) | Low | Deferred [S] |
| Theme polish: `#333` h1, gray input borders, UI emoji, inline handlers | Low | Deferred [S] |

**Enhancements:** size/margin/colour controls + SVG export [S]; copy-image-to-clipboard [S].

### text-diff.html
| Defect | Severity | Status |
|---|---|---|
| Positional pairing of changed blocks — one inserted line mid-block cascades everything after into "changed" | Medium | Deferred — alignment heuristic or word-diff [M] |
| No CDN-failure guard (`Diff` undefined throws on first compare) | Low | Deferred [XS] |
| No size guard — very large inputs freeze the tab | Low | Deferred [S] |

**Enhancements:** word-level intra-line highlighting [M]; ignore-whitespace/case toggles [S]; collapse-unchanged context folding [M]; unified view + patch export [M].

### markdown-editor.html
| Defect | Severity | Status |
|---|---|---|
| Sanitized links allowed `target` without `rel` → reverse tab-nabbing | Medium | ✅ Fixed — DOMPurify hook forces `rel="noopener noreferrer"` |
| Autosave quota failure is silent — UI still implies autosave is active | Medium | Deferred — visible autosave indicator [S] |
| No CDN-failure degradation (editor dead with no message) | Low | Deferred [S] |
| Restored content always flagged dirty (beforeunload nags with zero edits) | Low | Deferred [S] |

**Enhancements:** formatting toolbar + word count [M]; synced scroll [S].

### field-mapper.html
| Defect | Severity | Status |
|---|---|---|
| CSV formula injection — leading `= + - @` executed by Excel on export | High | ✅ Fixed — apostrophe-neutralized (plain negative numbers exempt) |
| Duplicate output-field names silently overwrite each other in exports/preview | Medium | Deferred — uniqueness guard [S] |
| Excel-serial date heuristic converts plain numbers (40000 → a 2009 date) via `date()` pipe | Medium | Deferred — explicit opt-in [S] |
| Only the first sheet is read; duplicate source headers collide | Low | Deferred — sheet picker [S] |
| Applying a preset silently replaces the current mapping | Low | Deferred — confirm dialog [XS] |
| Working session not persisted (only presets) | Low | Deferred [M] |

### dc-inventory.html — **OBSOLETE**
Marked obsolete per owner decision (July 2026): unmaintained, deliberately excluded from the hub, role-gated in routing. **No changes made or planned.** Known defects, documented for the record: non-transactional JSON import can wipe the DB permanently (a snapshot-before-import would fix it); base64-in-localStorage fallback can silently fail on >~5MB data; 13 columns rendered under a "12 quarters" spec; no Chart.js-failure degradation.

### disclaimer.html
No functional defects. It is the only page with dark-mode support — worth either extracting those tokens into DESIGN.md for all tools [L] or accepting the inconsistency.

---

## New-app candidates

Built this pass ✅:

| App | Notes |
|---|---|
| ✅ **JSON Formatter** (`json-formatter.html`) | Zero-dep: format/minify/validate with line/col errors, collapsible tree, search, path copy, stats, drag-drop |
| ✅ **Loan Calculator** (`loan-calculator.html`) | Integer-cents amortization, extra-payment comparison, per-year collapsible schedule, CSV export, Chart.js balance chart |

Remaining suggestions:

| Idea | Why it fits | Effort |
|---|---|---|
| Timestamp / epoch converter (unix ↔ ISO ↔ human, multi-tz) | Time cluster; pairs with the timezone board; zero deps | S |
| Image resizer & compressor (canvas, fully local) | Strongest privacy-first story; broad everyday utility | M |
| ICS invite generator (build `.ics` locally, tz-aware) | Complements timezone compare; no backend needed | S/M |
| Password / passphrase generator (`crypto.getRandomValues`, diceware) | Classic privacy-first utility; zero deps | S |
| Regex tester (live matches, groups, explanation) | Developer cluster with JSON formatter and text diff | M |

---

## Housekeeping

- ✅ README tool list updated (was 7 of 13 tools; now lists all live tools incl. the two new apps).
- ✅ `org-chart-guide.html` added and linked from the Org Chart header (Help) — use-case-driven how-to covering import, files/autosave, editing, navigation, filters, dotted lines, Compare, Merge, exports, pivot, data safety, and reference tables.
- ✅ CLAUDE.md updated: persistence table (`loanCalculatorState`), CDN list (qrcode-generator), file structure (guide page, dc-inventory obsolete).
- Site-wide CDN pin + SRI remains the top infrastructure suggestion [M].
