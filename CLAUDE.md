# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Personal trip-planning site for an Aug 19 – Sep 5, 2026 Portugal/Spain family trip (Porto → Lisbon → Barcelona → Zaragoza → Madrid, + Segovia day trip). Live at https://syavayki.github.io/euro-trip-2026/.

The user uses Claude to iteratively update the plan — adding restaurants, sights, day-by-day notes, logistics, etc. Treat requests like "add X to Porto food" or "we're skipping Y" as ongoing edits to the markdown files in `docs/instructions/`.

## Commands

```bash
npm start    # serves the site locally via `serve .` (devDependency, no npx)
npm run dev  # same, alias for start
```

Deployment is fully automated: push to `main` → `.github/workflows/pages.yml` uploads the repo root to GitHub Pages. There is no build step, no bundler, no tests, no linter.

## Architecture

This is a **single-file static site**. The entire application lives in `index.html`:

- Vanilla JS, no framework. `marked` (markdown → HTML) is loaded from a CDN.
- Trip content is authored as markdown in `docs/instructions/` and fetched at runtime, rendered into `#content`, and cached in-memory per file.
- The `TABS` object in the `<script>` block is the **single source of truth** mapping the city/sub-tab UI to markdown files. Adding a new city or sub-section means editing `TABS` *and* creating the corresponding `.md` file — nothing else. `TABS` also drives the "print entire guide" assembly order.
- The Overview tab gets special treatment: `injectTOC()` assigns slug IDs to headings, then `buildAccordions()` folds the master itinerary into nested collapsibles (level 1 = `##` city/global sections, level 2 = `###` days & reference blocks), then a sticky sidebar TOC is built (H2 groups with H3 "Day N" entries) with an `IntersectionObserver` highlight. Open/closed accordion state persists in `localStorage` (`accOpen:v1`); sections whose heading dates include *today* auto-expand with a "📍 Today" badge (`parseDateRange` reads "Aug 20–23" / "Day 1 — Thursday Aug 20" out of heading text, year hardcoded 2026). TOC clicks and `#hash` deep links expand collapsed ancestors via `revealSection()`. The TOC/accordions only activate for the overview file; other tabs render plain.
- Mobile-first chrome: both the city-tab bar and sub-tab bar are sticky (`--tab-h`/`--subt-h` govern all offsets, incl. `scroll-margin-top` on headings); a scroll-to-top FAB appears past 500px; 3+-column markdown tables render as tap-to-expand cards (`transformTables()`).
- Printing: the 🖨️ button in the sub-tab bar offers "print this tab" (`window.print()`) and "print entire guide" (`printEntireGuide()` fetches every `TABS` file into a hidden `#print-all` container and prints with `body.print-all-mode`). `@media print` force-opens all accordions/cards and page-breaks before each city/document.
- `.nojekyll` at the repo root is required so GitHub Pages serves the markdown files in `docs/` verbatim instead of running them through Jekyll.

## Content layout

```
docs/instructions/   # Markdown rendered by the site (one file per city × {food, sights}, plus master itinerary)
docs/goog-map-data/  # CSVs intended for import into Google Maps "My Maps" lists — NOT rendered by the site
```

### Editing trip-research markdown

**Document roles** (keep these straight — it's the core structure of the guide):
1. `euro_trip_2026.md` (master, "Overview" tab) = the super-detailed tour guide. Every day is executable from this file alone: times, routes, tickets, booking refs, fallbacks, local tips.
2. `{city}_overview.md` = compact snapshot (logistics, getting around, outline itinerary, quick-reference, must-book, budget). No full day-by-day here.
3. `{city}_food_research.md` / `{city}_sights_research.md` = broad reference catalogs of all options, not just the chosen plan.

**Formatting conventions:**
- Master heading shapes are load-bearing for the TOC + accordions: city sections are `## {flag} {City} — {dates} ({n} nights)`, days are `### Day N — {Weekday} {Mon} {D} · {Theme}` (also `Day N (Option A)` and `Day Trip` variants). Don't change without updating `buildTOC()`/`buildAccordions()` in `index.html`.
- Inside a day: bold time blocks (`**HH:MM — Thing**`), option tables, and blockquotes ONLY for tips/warnings/quotes (`> 💡 **Local tip:** …`, `> ⚠️ …`) — never wrap plain timeline lines in `>`.
- Tables with 3+ columns render as tap-to-expand cards: column 1 = name (bold), column 2 = short "what/why" summary line; put details in later columns. 2-column tables stay flat — use them only for key-value logistics.
- Booked facts (flight/train numbers, locators, confirmation numbers, PINs, paid ticket refs) are copied verbatim, never paraphrased. When facts conflict between files, the master wins.
- Weekday-vs-date errors have bitten before (e.g. "Thursday Sep 4" when it's a Friday). Double-check any weekday you write against the 2026 calendar.

### Editing CSVs in `docs/goog-map-data/`

These feed the user's custom Google "My Maps" — they are the map the user actually pulls up while traveling. **When a new location (restaurant, sight, bar, etc.) is added to any city's markdown file, also add it to the matching CSV in `docs/goog-map-data/`** (e.g. a new Porto restaurant goes into both `porto_food_research.md` and `porto_restaurants.csv`). Missing a CSV update means the spot won't appear on the map.

Past commits show recurring formatting pain: Google Maps' importer is fussy about delimiters and quoting. Current format is **comma-delimited with double-quoted fields** for any cell containing commas. If you change a CSV, preserve quoting on every multi-word/comma-containing cell or the import will silently split columns.

## Conventions

- This is a personal repo — no PRs, no review process. Commit directly to `main`; the deploy follows automatically.
- Keep `index.html` self-contained. Don't introduce a build tool, bundler, package manager dependencies, or split into multiple JS/CSS files unless there's a strong reason — the appeal of this project is that the entire app is one readable file.
- npm (not pnpm) per global instructions.
