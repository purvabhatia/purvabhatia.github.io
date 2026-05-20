# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Static personal portfolio for Purva Bhatia (MS Financial Analytics, SJSU), deployed via GitHub Pages. There is no build step, package manager, bundler, linter, formatter, or test runner — do not attempt to run any. Edit HTML/CSS/JS directly and push to `main`; GitHub Pages serves it.

To preview locally, open the HTML files directly in a browser (no dev server needed).

## Architecture

Three source files render the entire site. Understanding how they relate matters because the CSS is intentionally split and the JS is intentionally inline.

### CSS split rule (important)

- `style.css` is the **shared design system only**: `:root` color/font tokens, body reset, fixed nav, footer, and the two full-viewport overlays (`body::before` noise, `body::after` grid). Anything visible on every page lives here.
- Each HTML file owns a large `<style>` block for **page-specific** rules (hero, sections, project cards, mortgage calculator UI, etc.). `mortgage.html` also re-declares `:root` to add `--green` / `--yellow` / `--red` for affordability indicators.
- When changing visuals, decide first: shared (edit `style.css`) or page-local (edit the `<style>` block in that HTML file). Don't move page-specific rules into `style.css` — it would force every page to load CSS it doesn't use.

### `index.html` — portfolio

Single-page layout: hero → about → education → experience → projects → certifications → contact. Sections use class `reveal` and are progressively revealed by an `IntersectionObserver` defined at the bottom of the file (around line 995). The observer toggles the `visible` class at a 0.1 threshold — any new section that should fade in needs the `reveal` class plus a CSS rule that animates `.reveal.visible`.

Resume PDF and the Excel simulator are linked from `assets/`. The projects section links out to `mortgage.html` as the only interactive demo.

### `mortgage.html` — interactive sub-page

Self-contained mortgage affordability calculator. The script block (around line 994) is structured as three layers:

1. **Data tables** — `CREDIT_RATES` (FICO band → interest rate), `TAX_RATES` (state → property tax rate, all 50 states + DC), `INSURANCE` (state → annual home insurance $). These are hardcoded from the source Excel (`assets/Simulator.xlsx`); if rates change, update the constants here and re-upload the workbook.
2. **Helpers** — `fmt`, `fmtK` format currency; `pmt(rate, nper, pv)` is the standard mortgage payment formula; `buildAmort(...)` produces the amortization schedule rows.
3. **`calculate()`** — the single function that recomputes everything: pulls all input values, looks up rate/tax/insurance from the tables, computes P&I via `pmt`, sums monthly housing cost, derives the affordability index vs. monthly income, and writes results back to the DOM. Every input has an event listener that calls `calculate()`, and `calculate()` is invoked once on load.

When adding a new input, follow the same three-step pattern: add the field to the form, read it inside `calculate()`, and wire up an event listener at the bottom of the script.

### `assets/`

Headshot JPG, resume PDF, and Excel workbooks (`Simulator.xlsx` is the source for the mortgage calculator's constants; `Final_Costco_FP&A_Term Project.xlsx` is a downloadable project artifact).

## Conventions

- All JavaScript is inline `<script>` at the bottom of each HTML file. There are no external `.js` files and none should be added unless a real need emerges — the site is small enough that an extra HTTP request isn't worth it.
- Open Graph meta tags exist on both pages — keep them in sync with the page's actual title/description if content changes.
- Internal nav anchors (`#about`, `#projects`, etc.) live only on `index.html`. `mortgage.html` is a leaf page; if you add nav to it, account for the back-to-portfolio path (currently a known gap, see `TODO.md`).

## Related files

- `AGENTS.md` — short overview of the same constraints (kept for non-Claude tooling).
- `TODO.md` — prioritized backlog (P0 mobile nav fixes, P1 content tightening, etc.). Check this before suggesting feature work so you don't propose something already tracked.
