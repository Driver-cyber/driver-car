# Driver — Car Buying Tracker

A small, fast, mobile-first web app for tracking used-car listings during a
car search (Vancouver, WA). Built for two phones, no build step, deployed as
static files on **GitHub Pages**.

Started as a Kia **Telluride** tracker and architected with tabbed
**categories**. Now ships with two tabs: **Tellurides** and **Minivans**
(Sienna · Odyssey · Carnival · Pacifica). Adding another vehicle type is a
drop-in.

## Run it

It's a single static page — just open `index.html`:

```bash
# any static server works; e.g.
python3 -m http.server 8000
# then visit http://localhost:8000
```

Opening the file directly (`file://`) also works — the seed data is embedded
inline as a fallback for when `fetch()` is blocked.

## What's in here

| File | Purpose |
| --- | --- |
| `index.html` | The whole app — inline CSS/JS, no dependencies. |
| `seed-data.json` | Telluride seed/schema, loaded on first run. |
| `seed-minivan.json` | Minivan seed (Sienna/Odyssey/Carnival/Pacifica). |
| `CLAUDE.md` | Architecture + handoff notes. |

## Features

- Listing cards with price-confidence badges, favorites, source links
- Add / edit / delete listings
- Sort (price, mileage, year, best-value) + filter (trim, drivetrain, favorites)
- Per-car test-drive checklist
- Budget banner with over-target flagging
- Trim primer, known-issues reference, and search-hub quick links
- JSON export / import / reset-to-seed (state lives in `localStorage`)

## Deploy (GitHub Pages)

Settings → Pages → deploy from branch, root. No build step.

See `CLAUDE.md` for architecture, data model, and next steps.
