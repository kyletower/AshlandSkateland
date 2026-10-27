# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Skate RVA (Ashland Skateland) marketing site — a static, dependency-free website with no build step, no package.json, and no test suite. Every page is a single self-contained `index.html` with CSS and JS inlined in `<style>`/`<script>` tags. The root-level `script.js` and `styles.css` are currently empty/unused — all styling and behavior lives inline in each page.

## Running locally

There's no build or dev server tooling configured. Open the HTML files directly in a browser, or serve the directory with any static file server (e.g. `python3 -m http.server`) to test paths like `/calendar/` and `/menu/` that rely on clean (extensionless) URLs.

## Structure

- `index.html` — home page: hero, public skate schedule, party packages, private rentals, fundraiser calculator, lessons, reviews, contact/map.
- `calendar/index.html` — embeds `calendar/october.pdf` in an iframe. The PDF must be replaced monthly; there's no dynamic calendar logic.
- `menu/index.html` — snack bar menu with an allergen legend.
- `assets/skate-rva-logo.svg` — shared logo referenced via relative paths (`assets/...` from root, `../assets/...` from subpages).

Navigation between pages uses root-relative paths (`/`, `/calendar/`, `/menu/`), so the site must be deployed in a way that resolves these correctly (not from a subpath).

## Important duplicated data (keep in sync)

`index.html` has business data that is intentionally duplicated between the static HTML (what users read) and a `SCHEDULE`/`RENTAL_TIERS` JS structure (what the countdown timer and sliders compute from). Comments in the code already flag this, but when editing hours, prices, or rental tiers, update **both**:

- **Public Skate Schedule table** (`#schedule` section) ↔ the `SCHEDULE` array in the inline `<script>`. The array is indexed by `Date.getDay()` (0 = Sunday), and each entry is `{ start, end, price }` in 24-hour `"HH:MM"` time.
- **Private Building Rental** pricing ↔ the `RENTAL_TIERS` array, which drives the `#rental-guests` slider (slider value is an *index* into this array, not a guest count).
- **Private Fundraiser** formula (`0.8 * (guests * price - 375)`) is described in the on-page copy and implemented in `updateFundraiserSliders()` — if the deal terms change, update the paragraph text and the formula together.

## Conventions

- Dark neon theme via CSS custom properties defined per-page in `:root` (`--bg-black-magic`, `--neon-pink`, `--neon-cyan`, etc.). `calendar/index.html` and `menu/index.html` redefine a subset of the same variable names rather than sharing a stylesheet — keep values consistent across pages if the palette changes.
- Booking CTAs are `sms:` links with a prefilled body text (e.g. `sms:8047986550?&body=...`), not a form — there's no backend.
- All external links (Google/Facebook reviews, Google Forms for party/team booking) use `target="_blank" rel="noopener"`.
