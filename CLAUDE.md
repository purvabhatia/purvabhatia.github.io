# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Static personal portfolio for Purva Bhatia (MS Financial Analytics, SJSU), deployed via GitHub Pages. There is no build step, package manager, bundler, linter, formatter, or test runner — do not attempt to run any. Edit HTML/CSS/JS directly and push to `main`; GitHub Pages serves it.

To preview locally, open the HTML files directly in a browser (no dev server needed).

## Architecture

The site is **six hand-written source files** plus `assets/`: one shared stylesheet (`style.css`) and five self-contained HTML pages, each owning its own inline `<style>` and `<script>`. There are two kinds of page:

- **Portfolio** (`index.html`) — the hub, and the only page with internal section nav.
- **Leaf pages** (`mortgage.html`, `costco.html`, `3m.html`, `stocks.html`) — standalone project demos linked from the portfolio's projects section. Each has a shared `<nav>` (logo → `index.html`, "← Back to Projects" → `index.html#projects`) and no internal section anchors.

### CSS split rule (important)

- `style.css` is the **shared design system only**: `:root` color/font tokens (a **light** theme — white / slate surfaces, a **navy-blue** accent, near-black slate text, and **Inter** for all type), body reset, fixed nav, footer, and a faint full-viewport grid overlay (`body::after`). Anything visible on every page lives here.
- Each HTML file owns a large `<style>` block for **page-specific** rules (hero, sections, project cards, mortgage calculator UI, DCF tables/heatmaps/charts, etc.). The directional/semantic finance colors (`--green` / `--yellow` / `--red`, plus the DCF-scenario `--bull` / `--base` / `--bear`, where `--bull`/`--base`/`--bear` alias `--green`/`--accent2`/`--red`) live in the shared `style.css` `:root` — **no page re-declares `:root` locally** anymore.
- When changing visuals, decide first: shared (edit `style.css`) or page-local (edit the `<style>` block in that HTML file). Don't move page-specific rules into `style.css` — it would force every page to load CSS it doesn't use.

### `index.html` — portfolio

Single-page layout: hero → about → education → experience → projects → certifications → contact. The fixed `<nav>` carries a `.nav-brand` block — the name over an always-visible `.nav-contact` line (email · LinkedIn · phone). The **hero** is a two-column `.hero-grid` (headline + CTAs on the left; headshot and a four-tile `.stat-block` of credibility metrics on the right), and **About** is narrative paragraphs beside a skills column (`.about-right`) — the headshot and stat tiles live in the hero, not About. The **experience** entries use `.timeline-desc` bullet lists (`<ul>`), not paragraphs. Sections use class `reveal` and are progressively revealed by an `IntersectionObserver` (in the inline `<script>` near the bottom of the file) that toggles the `visible` class at a 0.1 threshold — any new section that should fade in needs the `reveal` class plus a CSS rule that animates `.reveal.visible`.

The **projects section** is a grid of six `.project-card`s. Each card leads with its title, then a `.project-stats` highlight row — big bold numbers with directional color (`.pstat-num.down` red for an overvaluation call, `.pstat-num.up` green for the live dashboard) — then a description and `.project-tag`s. Demo-backed cards (3M, Costco, mortgage, stocks) end with a `.project-actions` row: a primary "Live Demo ↗" to the leaf page plus an outline "Download …" to the source file in `assets/`. Description-only cards (Financial Metrics Dashboard, FP&A SQL) carry no buttons. There are **no emojis or "Live Demo" badges** — both were removed, so the button alone signals the demo; the leaf pages must not link to themselves (see the DCF section). The `.project-icon` / `.project-badge` / `.project-arrow` CSS rules are now unused leftovers.

### `mortgage.html` — interactive calculator

Self-contained mortgage affordability calculator. The script block is three layers:

1. **Data tables** (~line 996) — `CREDIT_RATES` (FICO band → interest rate), `TAX_RATES` (state → property tax rate, all 50 states + DC), `INSURANCE` (state → annual home insurance $). Hardcoded from `assets/Simulator.xlsx`; if rates change, update the constants here and re-upload the workbook.
2. **Helpers** — `fmt`, `fmtK` format currency; `pmt(rate, nper, pv)` is the standard mortgage payment formula; `buildAmort(...)` produces the amortization schedule rows.
3. **`calculate()`** (~line 1054) — the single function that recomputes everything: reads all inputs, looks up rate/tax/insurance from the tables, computes P&I via `pmt`, sums monthly housing cost, derives the affordability index vs. monthly income, and writes results back to the DOM. Every input has a listener that calls `calculate()`, and `calculate()` also runs once on load.

When adding a new input, follow the same three-step pattern: add the field to the form, read it inside `calculate()`, and wire up a listener at the bottom of the script.

### `costco.html` & `3m.html` — DCF valuation showcase pages

The most involved pages: single-page equity-research write-ups that visualize a discounted-cash-flow model. They share one structure — build a new one by copying the other:

- **~9 numbered sections** from `01 / Executive Summary` to `09 / Recommendation`, each a `<section class="reveal">` with `.section-label` / `.section-title` / `.section-sub`, plus a KPI strip near the top. Middle sections are model-specific: Costco has Revenue Forecast (ETS) and Working Capital & AFN; 3M has Litigation Modeling.
- **Charts via Chart.js 4.4.1** (CDN `<script>` in `<head>` — the only external runtime dependency on the site). Each chart is a `<canvas id="chartX">` in the body wired to a `new Chart(document.getElementById('chartX'), {...})` call in the inline `<script>` at the bottom. Non-chart widgets — the WACC × terminal-growth **sensitivity heatmap** (`#heatmap`) and the `#segmentList` — are built by hand in that same script.
- **Header** has the shared `<nav>` plus a `.header-actions` block with "↓ Download Excel Model" (primary, links to the workbook in `assets/`) and "← Back to Projects" (outline). **Do not add a "Live Demo" button here** — the page *is* the live demo; that affordance belongs on the `index.html` project card. A header self-link was already a bug fixed once.

**Every headline number, chart series, and sensitivity cell on these pages is derived from the page's source workbook and must reconcile to it** — see `assets/` below.

### `stocks.html` — embedded Tableau Public dashboard

A different kind of data page from the DCF showcases: instead of drawing its own charts, it **embeds a published Tableau Public viz** of a single-stock (NVDA) price dashboard. Same leaf-page shell (shared `<nav>`, hero header, KPI strip, numbered sections, footer), but:

- The live viz is a `<tableau-viz>` web component from the **Tableau Embedding API v3** (CDN `<script type="module">` in `<head>` — the site's *second* external runtime dependency, after Chart.js), pointing at the published view (`public.tableau.com/views/NVDAReal-TimeStockDashboard/NVDADashboard`) and framed in a light `.viz-frame` card so the dashboard's own light theme reads intentionally against the dark page.
- **No figures are hardcoded into the page.** The live numbers live inside the embedded viz; the KPI strip uses only *stable descriptors* (ticker, data window, source, tool) so it can't drift out of sync with the embed. This is the inverse of the DCF pages, where every number must reconcile to a workbook — here there is nothing on the page to reconcile.
- Data pipeline (see `assets/` below): a **private** Python script (not committed — it holds an Alpha Vantage API key) pulls NVDA daily OHLCV from the Alpha Vantage API → `assets/stock_data.csv` (tracked; the page's download) → the Tableau workbook `assets/NVDA_stock.twb` → published to Tableau Public as an **extract** (a snapshot) → embedded here. To refresh: re-run the script, refresh the workbook's extract, re-publish (the embed updates), and re-commit the CSV. Because the embed is a snapshot, the page labels its freshness in the `.asof` callout.

### `assets/` — source workbooks are the source of truth

Headshot, resume PDF, and the Excel workbooks that back the data-driven pages:

| Workbook | Backs | Drives |
|---|---|---|
| `Simulator.xlsx` | `mortgage.html` | the `CREDIT_RATES` / `TAX_RATES` / `INSURANCE` constants |
| `Costco_DCF.xlsx` | `costco.html` | every figure, chart series, sensitivity cell |
| `3m_DCF.xlsx` | `3m.html` | every figure, chart series, litigation schedule |

The workbook is the **source of truth and is treated as read-only** — the user edits it in Excel directly, then asks Claude to reconcile the HTML to match (not the other way around). When a workbook changes, update the page's hardcoded numbers and chart data to agree with it; when a page cites a value, it should trace to a cell. The same workbook is also offered as a download from the page, so the copy in `assets/` and the page must stay in sync. Audits of these models (formula/methodology correctness, three-statement reconciliation) are done out-of-band with `openpyxl` by opening the workbook twice — `data_only=True` for computed values and `data_only=False` for formulas — and comparing; never by modifying the workbook.

The **stock-dashboard pipeline is the exception to the reconcile rule** (see `stocks.html` above). `stock_data.csv` (NVDA daily OHLCV) is *regenerated* by the private Python fetch script — not hand-maintained — and is offered as the page's download; `NVDA_stock.twb` is the Tableau workbook published to Tableau Public. The `.twb` is plain XML, so it's audited by reading it directly, not via `openpyxl`. Neither is a hardcoded source for page figures — `stocks.html` embeds the published viz instead of re-stating its numbers — so don't try to reconcile page values to the CSV; there are none.

## Conventions

- All JavaScript is inline `<script>` at the bottom of each HTML file. There are no external `.js` files and none should be added unless a real need emerges — the site is small enough that an extra HTTP request isn't worth it. Two pages load an external library via CDN: **Chart.js** on the DCF pages, and the **Tableau Embedding API** on `stocks.html`.
- Open Graph meta tags exist on every page — keep them in sync with the page's actual title/description when content changes.
- Internal nav anchors (`#about`, `#projects`, …) live only on `index.html`; leaf pages link back via `index.html#projects`. Nav links are `display:none` on mobile with no hamburger replacement yet, so don't assume mobile nav works — it's a tracked P0 gap (see `TODO.md`).
- Keep the two DCF pages visually consistent: they deliberately share one design language, so port patterns between `costco.html` and `3m.html` rather than inventing new ones.

## Related files

- `AGENTS.md` — short overview of the same constraints (kept for non-Claude tooling); update it alongside this file when the architecture changes.
- `TODO.md` — prioritized backlog (P0 mobile nav, P1 content tightening, etc.). Check it before suggesting feature work so you don't propose something already tracked.
