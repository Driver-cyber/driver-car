# Driver — Car Buying Tracker · Claude handoff

A small, fast, mobile-first web app for Chad **and** Joelle to track used-car
listings during a search in Vancouver, WA. Started as a Kia Telluride tracker;
built to grow into other vehicle types (Minivans next) via tabbed **categories**.
Deployed as static files on **GitHub Pages**.

## Tech stack & constraints

- **Vanilla HTML/CSS/JS, no build step.** Single `index.html` (inline CSS/JS).
  Must work as static files on GitHub Pages (no server runtime).
- **State: `localStorage`** (`driver-car-state-v1`), seeded on first load from
  `seed-data.json`. The seed is also embedded inline in `index.html` as a
  fallback so the app runs from `file://` where `fetch()` is blocked.
- **JSON import/export** for backup and manual cross-phone sync.
- No frameworks.

## Architecture

State shape (one level above the original seed schema):

```
state = {
  schemaVersion, activeCategory, lastUpdated,
  categories: {
    telluride: { label, make, model, primerLabel, budget, financing, trimPrimer[],
                 checklistTemplate[], knownIssues{}, listings[], searchHubs[] },
    minivan:   { ...same shape... }   ← shipped
    suv:       { ...same shape... }   ← shipped (3-row SUVs)
    // add another category to CATEGORY_SEEDS → it gets a tab + full UI
  }
}
```

`seed-data.json` (Telluride), `seed-minivan.json` (Minivans), and
`seed-suv.json` (3-row SUVs) are the canonical per-category schema sources —
each a raw doc with top-level keys `budget`, `financing`, `trimPrimer`,
`checklistTemplate`, `knownIssues`, `listings[]`, `searchHubs[]`, plus an
optional `meta` block (`label`, `make`, `model`, `primerLabel`).

`CATEGORY_SEEDS` (in `index.html`) is the registry: each entry names a seed
file + an inline embedded fallback + default meta. `buildDefaultCategories()`
loads them and `categoryFromSeed()` wraps each raw doc into a category record.
On load, `init()` backfills any newly-registered category into an existing
saved state **without** clobbering the user's edits — so adding a category is
a true drop-in even for returning users.

For the Minivans and 3-Row SUVs categories, "trim" is repurposed to mean
**model** (Sienna / Odyssey / Carnival / Pacifica; Palisade / Ascent / Pilot)
so the trim filter filters by model; the trim level lives in each listing's
title.

Listings also carry `votes: [{ id, type:"up"|"down", note }]` (test-drive
feedback, editable/deletable, sortable by net score) and a `testDriven` flag
(toggle + "Driven only" filter).

The `store` module (get/save/clear over localStorage) is the **seam for Phase 2**
live sync (Cloudflare Worker + KV, or Firebase) — swap it without touching the UI.

## Features (shipped — Phase 1)

- Listings cards: title, year, trim, drivetrain, mileage, price + confidence
  badge (confirmed / estimate / unknown), dealer + distance, status, favorite
  star, source link ("Find at source", new tab).
- Add / edit / delete listings (all fields editable inline).
- Sort (price / mileage / year / best-value) + filter (trim, drivetrain,
  favorites-only).
- Per-car checklist from `checklistTemplate`, checked state saved per car.
- Budget banner: pre-tax ($22.5k) + all-in ($25k) + rate; over-target flagged.
- Collapsible trim primer (highlights the SX "7 seats / not 8" warning),
  known-issues reference, and search-hub quick-launch buttons.
- JSON export / import / reset-to-seed.

## Design system (Chad's house style)

- Headlines: **Fraunces**, italic. UI/labels/mono: **JetBrains Mono**.
- Earthy palette: rust, gold, bone, ink. Generous negative space. Cards, not a
  spreadsheet.

## Gotchas (don't lose these)

- **8 seats requires the bench.** SX captain's-chair cars are 7 seats — surfaced
  with a red warning in the trim primer.
- **Prices go stale fast.** Most seed listings are `price: null` / `unknown` by
  design — they need pulling from the source link. UI says "find at source"
  because the links are mostly search hubs, not per-VIN pages.
- Financing (4.99% CU pre-approval before the dealer) is reference context in
  the budget banner, not a built-out feature.

## Acceptance criteria (all met)

- Loads on a phone, seeded from `seed-data.json` (or inline fallback) on first run.
- Add a listing → persists across reloads.
- Check an item on one car → doesn't affect another car.
- Export → re-import → identical state.
- The 2020 LX (`lx-2020-84k`) shows favorited as the top target.

## Shipped — Phase 1.5

- **Minivans tab.** A `minivan` category covering **Sienna / Odyssey /
  Carnival / Pacifica**, with a per-model primer, minivan-specific checklist
  (sliding doors, hybrid battery, Stow 'n Go), known issues, and seed listings
  pulled from current Vancouver-WA-area pricing (mostly `estimate` confidence —
  verify at source). Top in-budget targets: high-miles **Odyssey EX-L** and a
  2022 **Carnival LX**.

## Shipped — Phase 1.6

- **3-Row SUVs tab.** A `suv` category covering **Hyundai Palisade /
  Subaru Ascent / Honda Pilot**, with a per-model primer (incl. the Palisade =
  Telluride-twin note and the 8-seat-bench-vs-captains trap), an SUV-specific
  checklist (CVT/9-speed checks, 3rd-row car-seat access), known issues, and
  seed listings from current Vancouver-WA pricing. In-budget targets favorited:
  a confirmed 2018 **Pilot EX-L** (~$21.5k) and a 2019 **Ascent Premium** (AWD).
- **Test-drive votes + Driven tag** on every card (all categories).

## Next

- Phase 2 live sync via the `store` seam (Cloudflare Worker + KV, or Firebase).
- Refresh prices (they go stale fast) and add real per-VIN listings as
  Chad/Joelle find them.
- Possible: more categories (trucks? wagons?) — same drop-in pattern.
