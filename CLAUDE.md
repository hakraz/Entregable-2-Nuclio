# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

SportPulse is a lead magnet automation project: a single-page HTML landing page that captures visitor data and feeds it into an N8N workflow that generates personalized AI sports articles and delivers them via email.

## Local development

No build step — all HTML/CSS/JS is inline in a single file. To preview locally:

```bash
python -m http.server 8000
# then open http://localhost:8000/SportPulse_LandingPage.html
```

To test the N8N webhook manually:

```bash
curl -X POST <WEBHOOK_URL> \
  -H "Content-Type: application/json" \
  -d '{"nombre":"Test","email":"test@example.com","deporte":"Fútbol","source":"sportpulse-form"}'
```

## Architecture

### Frontend (`SportPulse_LandingPage.html`)

Single self-contained file (HTML + inline `<style>` + inline `<script>`). The form at `#sportpulse-form` (line 571) currently has only visual feedback — the real N8N `fetch()` POST needs to be wired in `handleSubmit()` (line 621).

Form payload shape:
```json
{
  "nombre": "string",
  "email": "string",
  "deporte": "Fútbol | Fórmula 1 | NBA / Baloncesto | Tenis | MMA / UFC",
  "source": "sportpulse-form"
}
```

### N8N workflow (5 nodes, external)

```
Webhook trigger → Google Sheets (append lead) → AI model (generate article)
    → Google Docs (create doc) → Gmail (send link) + Slack (notify team)
```

Each node is worth 2 points in the rubric. Credentials (Google, Gmail, Slack, AI provider) live in the N8N cloud instance — never in the repo.

## Design system

CSS custom properties are defined at the top of `<style>` (`:root` block, line 9):

| Token | Value | Use |
|---|---|---|
| `--neon` | `#00FF87` | Primary accent, CTAs |
| `--bg-primary` | `#0a0a0f` | Page background |
| `--bg-card` | `#12121a` | Card backgrounds |
| `--text-secondary` | `#8a8a9a` | Subtitles, captions |

Responsive breakpoints: 768px (grid collapses) and 480px (padding/font adjustments). Scroll animations use `IntersectionObserver` — add the `reveal` class to any element that should fade in on scroll.

## Key conventions

- All content is in Spanish (es).
- The file is intentionally self-contained — do not extract CSS/JS into separate files.
- The `docs/` folder contains course reference PDFs; do not modify them.
- Remote: `https://github.com/hakraz/Entregable-2-Nuclio.git` (branch: `main`).
