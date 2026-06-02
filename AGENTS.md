# AGENTS.md

## Project

Static personal portfolio site deployed via GitHub Pages. No build step, no bundler, no package manager — edit HTML/CSS/JS directly.

## Structure

Five source files plus `assets/`:

- `style.css` — shared design system (`:root` variables, body, nav, footer, noise/grid overlays)
- `index.html` — main portfolio (hero, about, education, experience, projects, certifications, contact); the only page with internal section nav. Project cards link out to the leaf pages with a "Live Demo" button and an Excel download.
- `mortgage.html` — mortgage affordability calculator sub-page (data tables + `calculate()`); page-specific `<style>` with extra `:root` overrides (`--green`, `--yellow`, `--red`)
- `costco.html`, `3m.html` — DCF valuation showcase pages: single-page equity-research write-ups with Chart.js charts, a WACC × terminal-growth sensitivity heatmap, and a KPI strip
- Leaf pages (`mortgage`, `costco`, `3m`) share a `<nav>` linking back to `index.html#projects` and have no internal section anchors.
- `assets/` — headshot, resume PDF, and the Excel workbooks that back the data pages: `Simulator.xlsx` → `mortgage.html`, `Costco_DCF.xlsx` → `costco.html`, `3m_DCF.xlsx` → `3m.html`

## Key constraints

- **CSS is partially shared.** `style.css` holds the common design system and base layout. Each HTML file retains a `<style>` block for page-specific rules. Changes to shared styles (colors, fonts, nav, footer, overlays) go in `style.css`; page-specific changes stay inline.
- **JS is inline.** All JavaScript lives in `<script>` blocks within each HTML file. No external JS files. The one exception is Chart.js, CDN-loaded on the DCF pages only.
- **Workbooks are the source of truth.** Each Excel file in `assets/` backs a page; the displayed numbers and constants are derived from it and must reconcile to it. The workbook is edited in Excel and treated as read-only by tooling — reconcile the HTML to the workbook, not the reverse. The same file is also offered as a download, so keep `assets/` and the page in sync.
- **No tooling.** There is no `package.json`, linter, formatter, typechecker, or test runner. Do not attempt to run any.
- **Deployment.** Push to `main` → GitHub Pages serves it. No preview or build step needed.
