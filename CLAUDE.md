# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Static multi-page marketing site for **certxpro.net** — Microsoft 900-series certification prep courses (AB-900, AZ-900, MS-900, PL-900) with newsletter (M365news) as primary conversion path. No build step; plain HTML. All CSS and JS are inlined inside each HTML file.

## Local development

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

Opening via `file://` works too; Python server avoids CORS quirks.

## Architecture

Each page is a **self-contained HTML file** — no external CSS or JS files. Every page carries:

1. `<style>` block in `<head>` — shared design tokens + page-specific styles
2. First `<script>` block before `</body>` — shared chrome functions (`renderNav`, `renderTicker`, `renderFooter`, `mountChrome`)
3. Second `<script>` block — page-specific logic (ticker init, form handlers, filter chips)

| Page | Role |
|---|---|
| [index.html](index.html) | Home — hero, ticker, path/steps, course grid, pull quote, CTA form → N8N |
| [courses.html](courses.html) | Course catalog (AB-900 first, newest→oldest), filter chips, bundle offer |
| [newsletter.html](newsletter.html) | M365news signup with envelope preview, dual forms → N8N |
| [about.html](about.html) | Portrait placeholder, bio, principles, FAQ accordion, contact band |

### Chrome injection

`mountChrome(activeKey)` inserts `.grid-bg` + `<nav>` at `afterbegin` and `<footer>` at `beforeend`. Valid `activeKey` values: `'home'`, `'courses'`, `'newsletter'`, `'about'`.

**The chrome functions (`renderNav`, `renderTicker`, `renderFooter`, `mountChrome`) are duplicated verbatim in all four HTML files.** Editing nav links, footer copy, or ticker items requires the same change in each file.

Each page must contain `<div id="ticker-slot"></div>` at the desired ticker position. Page JS replaces it:
```js
document.getElementById('ticker-slot').outerHTML = renderTicker();
```
`renderTicker()` duplicates the items array for CSS infinite-scroll.

### Webhook integration

Form handlers POST the following JSON:

```json
{ "email": "user@example.com", "exam": "...", "source": "..." }
```

| Page | Webhook URL | `exam` | `source` |
|---|---|---|---|
| `index.html` | `https://andrea-nuclio.app.n8n.cloud/webhook/649466da-d808-43cf-91c3-08a7edb25eb5` | `"Deliverable 2 Signup"` | `"home-cta"` |
| `newsletter.html` | `https://n8n.eu8.es/webhook/31b2ea6b-85ef-4f01-8de9-5b6d48000195` | `"M365news Newsletter"` | `"newsletter-page"` |

UI states: idle → `"Sending..."` (disabled) → `"✓ Subscribed"` or `"✕ Error, retry"` (re-enabled).

## Design tokens

All tokens live in the `:root` block inside each page's `<style>`. Accent is **emerald** (`--accent-a: oklch(0.78 0.14 165)` / `--accent-b: oklch(0.70 0.15 190)`). Gradients always `linear-gradient(135deg, var(--accent-a), var(--accent-b))`. Single responsive breakpoint at **860px**. Default content max-width **1240px** (`.wrap`).

Typography: Space Grotesk (display/body) + JetBrains Mono (labels/metadata) + Instrument Serif italic (editorial accents via `<em>`).

Body background glow: `radial-gradient(ellipse 110% 50% at 50% -10%, oklch(0.30 0.08 260 / 0.4), transparent 55%)`. Ellipse width must stay ≥ 110% — narrower widths make the fade edge align with content column edges.

## Key conventions

- All CSS and JS live **inline** inside each HTML file — no external files, no `assets/` folder.
- When editing styles or scripts, find them in the relevant HTML file's `<style>` or `<script>` block.
- Brand name is always **certxpro.net** (with domain) — never bare `certxpro` in headings or body copy.
- Cert names use a hyphen: `AZ-900`, not `AZ/900`. The `.slash` span carries `color: var(--fg-3)` for the dim hyphen.
- Courses catalog uses `.courses-wide` (max-width 1440px) instead of `.wrap` (1240px).
- Filter chips and course rows in `courses.html` are paired via `data-category` (`azure` / `m365` / `power` / `copilot`).
- Course order: newest to oldest — AB-900 (Apr 2026), AZ-900 (Mar 2026), MS-900 (Feb 2026), PL-900 (Jan 2026).
- Udemy links are placeholder `https://udemy.com` — replace before deploying.
- Footer must keep "Not affiliated with Microsoft Corporation."

## Reference docs (project root)

| File | Purpose |
|---|---|
| `N8N_WORKFLOW_SPEC_EN.md` | Full N8N workflow spec: 5-node pipeline (Webhook → Sheets → AI → Docs → Gmail + Slack), variable mapping, curl test commands |
| `WEB_UPDATES_MINIMAL_EN.md` | Exact JS code for the two form handlers + integration checklist. **Note:** the doc refers to `assets/js/index.js` / `assets/js/newsletter.js` which don't exist — apply those changes to the inline `<script>` blocks in `index.html` and `newsletter.html` respectively. |
| `PLAN_DELIVERABLE2_M3_MINIMAL.md` | Project overview and delivery scope |
| `docs/` | Module 3 course PDFs (funnels, content AI, automation, QA) — reference material only, not deployed |
