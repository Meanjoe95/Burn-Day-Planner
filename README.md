# Burn Day Planner

A single-file, self-hosted tool for prescribed fire planning: live NWS fire weather,
an FFS-guideline go/no-go call, and an FDACS-style smoke screening map — all in one
page, updated in real time, no installation required.

Built for Florida Forest Service prescribed burn planning. Not an official FDACS or
NWS product — see **Limitations & assumptions** below before relying on it operationally.

---

## What it does

- **Pin-drop location** — click anywhere on the map to set your burn point. Works
  anywhere in the continental US covered by NWS, not just one fixed unit.
- **Live fire weather** — pulls directly from `api.weather.gov`'s raw gridded forecast
  (the same underlying source as NWS's digital Fire Weather Planning Forecast), no
  scraping, no manual paste-in.
- **Automatic go / no-go call** — evaluates the pulled forecast against the FFS 2023
  Prescribed Fire Guidelines, including the "no more than 2 parameters outside
  guidelines" approval rule.
- **7-day scan** — tap through the week; weather, status, and the smoke cone all
  update together.
- **FDACS-style smoke screening cone** — a directional plume wedge (critical zone +
  smoke-sensitive zone) drawn on a real satellite/street map, sized by fuel type and
  ignition method, pointed downwind from the day's forecast transport wind.
- **Full hourly forecast table** — every pulled parameter, every hour, for the
  selected day.
- **Print-ready output** — one button prints just the smoke screening map (for
  attaching to a burn prescription), another prints the full 7-day hourly table.
- **Quick links** — Request a Spot Weather Forecast, and the NWS Fire Weather
  Forecast (FWF) text product for whichever office covers your current pin.

---

## Hosting it (GitHub Pages)

This is one self-contained HTML file — no build step, no server, no dependencies to
install. To make it usable from a phone or any browser:

1. Create a GitHub repo, upload this file, **rename it to `index.html`**.
2. Repo **Settings → Pages** → Source: *Deploy from a branch* → branch `main`, folder
   `/ (root)` → Save.
3. Wait ~1 minute, then open the URL GitHub gives you.
4. On iOS: open that URL in Safari → Share → **Add to Home Screen** for an app-like icon.

To update later: re-upload the new file as `index.html` the same way (Add file →
Upload files → Commit). The live site rebuilds automatically within a minute or two.

---

## Using it

### Top inputs
| Field | Source | Notes |
|---|---|---|
| Canopy type | Manual | Open/thinned = 95°F max temp; Closed/unthinned = 90°F max temp, per FFS guidelines |
| KBDI | Manual | Not available from any NWS feed — check FDACS's KBDI report and enter it yourself |
| Days since ¼" rain | Manual | Feeds the KBDI 401–500 band's rainfall-recency check |
| Fine fuel moisture (%) | Manual | Not available from NWS — from tables or direct measurement |

### Smoke screening panel
- **Fuel type** and **ignition method** dropdowns resize the critical zone (red) and
  smoke-sensitive zone (yellow) rings. Heavier fuels (slash > shrubs > litter > grass)
  and backing fire (weakest smoke-column lofting) both widen the rings; heading and
  ring ignition (strongest convection columns) tighten them. This is a planning
  heuristic based on general fire behavior principles — **not** FDACS's actual Simple
  Smoke Screening Tool calculation. Cross-check with that tool, VSmoke-Web, or a test
  fire before relying on it.
- Click the map to move the burn point; satellite/street toggle top-right.
- The cone direction follows the **selected day's** forecast transport wind — flip
  through days to see how the plume direction changes.

### Printing
- **Print smoke screen**: map + legend + location/date header, sized for attaching to
  a burn prescription.
- **Print 7-day forecast**: all 7 days' hourly data, one table per day. Choose
  **Landscape** in the print dialog for the widest layout.

---

## Data sources

| Data | Source | Live? |
|---|---|---|
| Hourly fire weather (wind, transport wind, mixing height, dispersion index, LVORI, temp, RH, sky cover, precip, visibility) | `api.weather.gov` gridded forecast | Yes |
| FFS 2023 threshold values | Florida Forest Service Prescribed Fire Guidelines for FFS Conducted Burning — 2023 | Hardcoded from the source document |
| Smoke screening distances (basic method) | UGA Guidebook for Prescribed Burning in the Southern Region, Ch. 6 | Hardcoded planning estimate |
| Fuel type / ignition method categories | Southern Fire Exchange factsheet describing FDACS's Simple Smoke Screening Tool | Reference only — not FDACS's live calculation |
| KBDI, fine fuel moisture | — | Manual entry (no live source found; FDACS's KBDI report doesn't expose a usable API) |
| Map tiles | OpenStreetMap (street) / Esri World Imagery (satellite) | Yes |

---

## Limitations & assumptions

- **Some NWS offices don't report every field.** If dispersion index, LVORI, or 20ft
  wind comes back empty, the tile shows "N/A" and that parameter is excluded from the
  go/no-go math (never silently treated as a pass or fail) — a warning banner explains
  which field is missing.
- **20ft wind and surface wind are treated interchangeably** for the go/no-go check,
  per FFS practice. If an office doesn't report 20ft wind, surface wind is used as a
  fallback automatically.
- **Fuel type / ignition method radii are a heuristic**, not FDACS's official tool
  output — see the note under the smoke screening panel.
- **Precip duration/begin/end are derived**, not a native NWS field — calculated from
  the first/last hour with active precipitation in the raw data.
- **"Chance of thunder" shows NWS's coverage term** (e.g. "Chance," "Likely") rather
  than a fabricated percentage, since the raw grid data doesn't expose an exact number.
- **This tool makes live calls to three external services** (NWS, and two map tile
  providers) every time it loads. If your network locks down outbound requests more
  tightly than a typical browser, it may fail where a single-connection tool wouldn't.
- Not a substitute for the day-of Spot Weather Forecast FFS policy requires — use the
  link at the bottom of the page to request one.

---

## Version notes

Current version reflects (in order): pin-drop location + live fire weather; full
hourly + 7-day tabular forecast; print support for the smoke map and 7-day table;
fuel type / ignition method smoke cone sizing; canopy-type temperature threshold
restored to the go/no-go logic; KBDI 401–500 band corrected to read "No Restriction"
(not "Manager Approval") when the rainfall-recency condition is met, per a close
re-read of the FFS 2023 guideline text; ignition method smoke-lofting direction
corrected for ring ignition (tightened, not widened) based on fire behavior literature.
