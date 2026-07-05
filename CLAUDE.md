# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Personal trip-planning site for an Aug–Sep 2026 Portugal/Spain trip (Porto → Lisbon → Barcelona → Madrid). Live at https://syavayki.github.io/euro-trip-2026/.

The user uses Claude to iteratively update the plan — adding restaurants, sights, day-by-day notes, logistics, etc. Treat requests like "add X to Porto food" or "we're skipping Y" as ongoing edits to the markdown files in `docs/instructions/`.

## Commands

```bash
npm start    # serves the site locally via `npx serve .`
npm run dev  # same, alias for start
```

Deployment is fully automated: push to `main` → `.github/workflows/pages.yml` uploads the repo root to GitHub Pages. There is no build step, no bundler, no tests, no linter.

## Architecture

This is a **single-file static site**. The entire application lives in `index.html`:

- Vanilla JS, no framework. `marked` (markdown → HTML) is loaded from a CDN.
- Trip content is authored as markdown in `docs/instructions/` and fetched at runtime, rendered into `#content`, and cached in-memory per file.
- The `TABS` object near the top of the `<script>` block (around line 295) is the **single source of truth** mapping the city/sub-tab UI to markdown files. Adding a new city or sub-section means editing `TABS` *and* creating the corresponding `.md` file — nothing else.
- The Overview tab gets special treatment: `injectTOC()` switches `#content` into a two-column layout, assigns slug IDs to `<h2>`/`<h3>` headings in the master itinerary, builds a sticky sidebar TOC (grouped by H2 city section with H3 "Day N" entries nested under it), and uses an `IntersectionObserver` to highlight the active section on scroll. The TOC only activates for the overview file; other tabs render plain.
- `.nojekyll` at the repo root is required so GitHub Pages serves the markdown files in `docs/` verbatim instead of running them through Jekyll.

## Content layout

```
docs/instructions/   # Markdown rendered by the site (one file per city × {food, sights}, plus master itinerary)
docs/goog-map-data/  # CSVs intended for import into Google Maps "My Maps" lists — NOT rendered by the site
```

### Editing trip-research markdown

- The master itinerary (`euro_trip_2026.md`) is the only file with the TOC sidebar. For the TOC to populate correctly, each city section must start with an `## H2` and each day must be an `### H3` beginning with `Day N` (e.g. `### Day 1 — Arrival`). Don't change this convention without also updating `buildTOC()` in `index.html`.
- City research files use markdown tables heavily. The site wraps every `<table>` in a `.table-wrap` div for horizontal scroll on mobile (`wrapTables()`), so wide tables are fine.

### Editing CSVs in `docs/goog-map-data/`

These feed the user's custom Google "My Maps" — they are the map the user actually pulls up while traveling. **When a new location (restaurant, sight, bar, etc.) is added to any city's markdown file, also add it to the matching CSV in `docs/goog-map-data/`** (e.g. a new Porto restaurant goes into both `porto_food_research.md` and `porto_restaurants.csv`). Missing a CSV update means the spot won't appear on the map.

Past commits show recurring formatting pain: Google Maps' importer is fussy about delimiters and quoting. Current format is **comma-delimited with double-quoted fields** for any cell containing commas. If you change a CSV, preserve quoting on every multi-word/comma-containing cell or the import will silently split columns.

## Conventions

- This is a personal repo — no PRs, no review process. Commit directly to `main`; the deploy follows automatically.
- Keep `index.html` self-contained. Don't introduce a build tool, bundler, package manager dependencies, or split into multiple JS/CSS files unless there's a strong reason — the appeal of this project is that the entire app is one readable file.
- npm (not pnpm) per global instructions.
