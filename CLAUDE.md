# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Offerts** is a Swedish PWA for tradespeople (snickare, elektriker, rörmokare, målare) to generate professional quotes on-site in 60 seconds. Deployed on Netlify as a static SPA.

## Architecture

This is a **single-file application** — all HTML, CSS, and JavaScript lives in `index.html`. There is no build step, no bundler, no npm, no framework. To make changes: edit `index.html` directly and commit.

### External dependencies (CDN, no local install)
- `html2pdf.js` — PDF generation
- Google Fonts (Instrument Serif, Syne, JetBrains Mono)

### Backend: Supabase (no SDK — raw fetch)
The app communicates with Supabase via direct `fetch()` calls, not the Supabase JS SDK. Auth tokens are stored in `localStorage` (`kly_token`, `kly_refresh`). All DB helpers are in the `/* SUPABASE HELPERS */` block (~line 1370).

Key Supabase tables: `quotes`, `customers`, `profiles`.

### State management
All state is module-level JS variables at the top of the `<script>` block:
- `quotes`, `customers`, `settings` — loaded from `localStorage`, synced to Supabase on write
- `user`, `supaUser` — auth state
- `selectedTrade`, `selectedProject`, `fieldValues`, `jobPhotos` — quote wizard state
- `priceList` — user's custom price register, stored in `localStorage` as `kly_prices`

localStorage keys: `kly_quotes`, `kly_customers`, `kly_settings`, `kly_logo`, `kly_prices`, `kly_token`, `kly_refresh`, `kly_user`, `kly_theme`, `kly_onboarded`, `kly_timerlog`.

### Screen navigation
Screens are `<div class="screen" id="scr-*">` elements. `showScreen(name)` activates one by adding the `active` class. Special full-screen overlays (login, onboarding, landing) use `position:fixed` and `display:flex/none` directly.

Active screens: `home`, `quotes`, `customers`, `stats`, `calc`, `detail`, `settings`, `pdf-preview`, `share`, `timer`, `pricing`.

### Quote wizard flow
`startNewQuote()` / `quickStart(trade, projId)` → step 1 (trade picker) → step 2 (project template) → step 3 (calc inputs) → step 4 (customer info) → `saveAndDone()`.

`goStep(n)` shows/hides `#step1`–`#step4` divs and calls `renderCalcInputs()` and `renderBreakdown()`.

### Pricing data
`PROJECTS` (const) — nested object keyed by trade (`bygg`, `el`, `vvs`, `mark`, `maleri`), each an array of project templates with `fields[]` arrays.

`PRICE_PRESETS` (const) — default prices per category, overridden by user's `priceList`.

`PROFESSIONS` (const) — array of profession objects, each with `trades[]` and `projects[]` arrays.

`PLANS` (const) — **must be defined** with keys `starter`, `pro`, `enterprise` each having `{ name, quotesPerMonth, users }`. This is referenced by `updatePlanUI()`, `renderQuotaBar()`, `selectPlan()`.

### AI features
`generateAIDescription(q)` and `importFromURL()` call `https://api.anthropic.com/v1/messages` directly from the browser. These require a valid `x-api-key` header — currently missing, so AI features silently fail and return `null`.

## Known structural issues to be aware of

- **`PLANS` constant is missing** — never defined but referenced everywhere. Must be added near the top of the `<script>` block.
- **`startOnboarding()` doesn't exist** — `checkOnboarding()` calls it; the correct function name is `showOnboarding()`.
- **Anthropic API calls lack an API key header** — both AI functions will get 401 errors silently.
- The `manifest.json` still uses the old name "Kalkylo" instead of "Offerts".

## Deployment

Push to `master` → Netlify auto-deploys. `netlify.toml` redirects all routes to `index.html` (SPA mode) and sets no-cache headers.

## Development workflow

Open `index.html` directly in a browser (file://) or serve with any static server:
```
npx serve .
# or
python -m http.server 8080
```

No compilation, no linting tools, no test suite. Changes are visible immediately on browser refresh.

## Conventions

- Helper shorthand: `const S = id => document.getElementById(id)` — used everywhere instead of `getElementById`.
- Currency formatting: always use `fmtSEK(n)` (uses `Intl.NumberFormat` with sv-SE locale).
- Toast notifications: `showToast(msg)` — auto-hides after 2.4s.
- All user-visible text is in Swedish.
- The app supports dark mode (default) and light mode via `body.light` class + CSS variables.
- CSS variables for theming are defined in `:root` and overridden in `body.light`.
