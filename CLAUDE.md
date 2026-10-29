# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Ashland Skateland marketing site — a static, dependency-free website with no build step, no package.json, and no test suite. Every page is a single self-contained `index.html` with CSS and JS inlined in `<style>`/`<script>` tags. The root-level `script.js` and `styles.css` are currently empty/unused — all styling and behavior lives inline in each page.

## Running locally

There's no build or dev server tooling configured. Open the HTML files directly in a browser, or serve the directory with any static file server (e.g. `python3 -m http.server`) to test paths like `/calendar/` and `/menu/` that rely on clean (extensionless) URLs.

## Structure

- `index.html` — home page: hero, public skate schedule, party packages, private rentals, fundraiser calculator, lessons, reviews, contact/map.
- `calendar/index.html` — data-driven monthly skate calendar rendered client-side from a `CALENDAR`/`WEEKLY_SCHEDULE`/`OVERRIDES` config in its inline `<script>` (see below). Includes a print stylesheet and a "Print / Save as PDF" button so staff can produce a printable monthly calendar without any build step.
- `menu/index.html` — snack bar menu with an allergen legend.
- `assets/skate-rva-logo.svg` — shared logo referenced via relative paths (`assets/...` from root, `../assets/...` from subpages).

Navigation between pages uses root-relative paths (`/`, `/calendar/`, `/menu/`), so the site must be deployed in a way that resolves these correctly (not from a subpath).

## Important duplicated data (keep in sync)

`index.html` has business data that is intentionally duplicated between the static HTML (what users read) and a `SCHEDULE`/`RENTAL_TIERS` JS structure (what the countdown timer and sliders compute from). Comments in the code already flag this, but when editing hours, prices, or rental tiers, update **both**:

- **Public Skate Schedule table** (`#schedule` section) ↔ the `SCHEDULE` array in the inline `<script>`. The array is indexed by `Date.getDay()` (0 = Sunday), and each entry is `{ start, end, price }` in 24-hour `"HH:MM"` time.
- **Private Building Rental** pricing ↔ the `RENTAL_TIERS` array, which drives the `#rental-guests` slider (slider value is an *index* into this array, not a guest count).
- **Private Fundraiser** formula (`0.8 * (guests * price - 375)`) is described in the on-page copy and implemented in `updateFundraiserSliders()` — if the deal terms change, update the paragraph text and the formula together.
- **`calendar/index.html`'s `WEEKLY_SCHEDULE`** array is a third copy of the same recurring public-skate hours (with per-session labels added for display), separate from `index.html`'s `SCHEDULE`/table — keep all three in sync when hours change.

## Updating the monthly calendar

`calendar/index.html` publishes one month at a time, entirely from the config block at the top of its `<script>`:

- `CALENDAR.year` / `CALENDAR.month` (0-indexed) select which month renders — change these to roll over to a new month.
- `WEEKLY_SCHEDULE` is the recurring weekly pattern (mirrors the home page schedule, see above).
- `OVERRIDES` is a `{ "YYYY-MM-DD": ... }` map of one-off exceptions for that month: `{ closed: true, reason, note }` for a closure, or `{ sessions: [...] }` to fully replace a day's normal sessions (e.g. a themed skate night). Clear out the old month's overrides when rolling to a new month.

The page builds the grid in JS (`renderCalendar()`), so there's no PDF to regenerate or replace. The print stylesheet (`@media print`) renders a light, ink-friendly single landscape page — if that stops fitting on one page after a content change, tighten spacing/font-size inside the `@media print` block rather than changing page margins first.

## Conventions

- Dark neon theme via CSS custom properties defined per-page in `:root` (`--bg-black-magic`, `--neon-pink`, `--neon-cyan`, etc.). `calendar/index.html` and `menu/index.html` redefine a subset of the same variable names rather than sharing a stylesheet — keep values consistent across pages if the palette changes.
- Booking CTAs are `sms:` links with a prefilled body text (e.g. `sms:8047986550?&body=...`), not a form — there's no backend.
- All external links (Google/Facebook reviews, Google Forms for party/team booking) use `target="_blank" rel="noopener"`.
