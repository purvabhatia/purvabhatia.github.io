# AGENTS.md

## Project

Static personal portfolio site deployed via GitHub Pages. No build step, no bundler, no package manager — edit HTML/CSS/JS directly.

## Structure

- `style.css` — shared design system (`:root` variables, body, nav, footer, noise/grid overlays)
- `index.html` — main portfolio (hero, about, education, experience, projects, certifications, contact); page-specific `<style>` for sections not in `style.css`
- `mortgage.html` — mortgage affordability calculator sub-page (linked from projects); page-specific `<style>` with extra `:root` overrides (`--green`, `--yellow`, `--red`)
- `assets/` — headshot, resume PDF, and Excel simulator source (`Simulator.xlsx`)

## Key constraints

- **CSS is partially shared.** `style.css` holds the common design system and base layout. Each HTML file retains a `<style>` block for page-specific rules. Changes to shared styles (colors, fonts, nav, footer, overlays) go in `style.css`; page-specific changes stay inline.
- **JS is inline.** All JavaScript lives in `<script>` blocks within each HTML file. No external JS files.
- **No tooling.** There is no `package.json`, linter, formatter, typechecker, or test runner. Do not attempt to run any.
- **Deployment.** Push to `main` → GitHub Pages serves it. No preview or build step needed.
