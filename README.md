# Burn Day Planner

A single-file, self-hosted tool for prescribed fire planning: live NWS fire weather,
an FFS-guideline go/no-go call, and an FDACS-style smoke screening map — all in one
page, updated in real time, no installation required, no dependencies to build.

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
  update together. Sized to fit its 7 days and centered above the map on desktop;
  reflows to fill the screen width on mobile so nothing gets clipped.
- **Smoke screening cone** — a directional plume wedge (critical zone + smoke-sensitive
  zone) drawn on a real satellite/street map, sized by fuel type and ignition method,
  pointed downwind from the selected day's forecast transport wind.
- **Full hourly forecast table** — every pulled parameter, every hour, for the
  selected day.
- **Print-ready output** — one button prints just the smoke screening map (for
  attaching to a burn prescription), another prints the full 7-day hourly table.
- **Quick links** — Request a Spot Weather Forecast, and the NWS Fire Weather
  Forecast (FWF) text product for whichever office covers your current pin.
- **Responsive** — fits phone screens down to ~360px wide without horizontal
  scrolling; the map, tiles, and inputs page reflow automatically at every width.
- **Degrades gracefully** — if the map library can't load (blocked network, offline
  CDN), weather data and the go/no-go call still work; only the map itself is affected.

---

## Using it

### Top inputs
| Field | Source | Notes |
|---|---|---|
| Canopy type | Manual | Open/thinned = 95°F max temp; Closed/unthinned = 90°F max temp, per FFS guidelines |
| KBDI | Manual | Not available from any NWS feed — check FDACS's KBDI report and enter it yourself |
| Days since ¼" rain | Manual | Feeds the KBDI 401–500 band's rainfall-recency check |
| Fine fuel moisture (%) | Manual | Not available from NWS — from tables or direct measurement |

### Weather tiles
Wind (20ft), Transport wind, Mixing height, Dispersion index, Temperature, Relative
humidity, Fine fuel moisture, KBDI, LVORI, Cloud cover, Chance of precip, and Precip
amount. The first nine are evaluated against FFS thresholds and color-coded
(green/amber/red); the last three are informational only (blue), since they're not
FFS go/no-go parameters. Visibility is tracked too but only shown in the hourly table,
not as a top tile.

Wind (20ft) is what drives the go/no-go wind check — FFS treats 20ft and surface wind
interchangeably, so surface wind is used automatically as a fallback if an office
doesn't report 20ft wind.

### Smoke screening panel
- **Fuel type** and **ignition method** dropdowns resize the critical zone (red) and
  smoke-sensitive zone (yellow) rings. Heavier fuels (slash > shrubs > litter > grass)
  widen the rings — more fuel, more smoldering, more smoke. Ignition method affects
  the rings differently: it's about how well the fire's convection column lofts smoke
  away from the ground, not how much smoke is produced. Backing fire has the weakest
  column of any technique, so its smoke lingers near the ground and gets a wider ring;
  heading and ring ignition both build strong columns that loft smoke up and away, so
  they get tighter rings; flanking sits in between. This is a planning heuristic based
  on general fire behavior principles — **not** FDACS's actual Simple Smoke Screening
  Tool calculation. Cross-check with that tool, VSmoke-Web, or a test fire before
  relying on it.
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
| FWF text product link | `forecast.weather.gov/product.php`, keyed to the NWS office covering the current pin | Yes |

---

## Limitations & assumptions

- **Some NWS offices don't report every field.** If dispersion index, LVORI, or 20ft
  wind comes back empty, the tile shows "N/A" and that parameter is excluded from the
  go/no-go math (never silently treated as a pass or fail) — a warning banner explains
  which field is missing.
- **Fuel type / ignition method radii are a heuristic**, not FDACS's official tool
  output — see the note under the smoke screening panel.
- **Precip duration/begin/end are derived**, not a native NWS field — calculated from
  the first/last hour with active precipitation in the raw data.
- **"Chance of thunder" shows NWS's coverage term** (e.g. "Chance," "Likely") rather
  than a fabricated percentage, since the raw grid data doesn't expose an exact number.
- **This tool makes live calls to external services** (NWS, and two map tile
  providers) every time it loads. If your network locks down outbound requests more
  tightly than a typical browser, weather data may still work while the map doesn't —
  it won't take the whole page down.
- Not a substitute for the day-of Spot Weather Forecast FFS policy requires — use the
  link at the bottom of the page to request one.

---

## Version notes

Most recent changes: fixed a mobile layout bug where the 7-day scan row could overflow
and drag the whole page into horizontal scroll on phone-width screens — now reflows to
fit any width from ~360px up; fixed a related resilience bug where a failed map-library
load could silently break weather data and the go/no-go call along with it — the two
are now decoupled; corrected the KBDI 401–500 band to read "No Restriction" (not
"Manager Approval") when the rainfall-recency condition is met, per a close re-read of
the FFS 2023 guideline text; corrected the ignition-method smoke-lofting direction for
ring ignition (tightened, not widened) based on fire behavior literature; restored the
canopy-type dropdown to the go/no-go logic; replaced the surface wind tile with 20ft
wind (used interchangeably per FFS practice, with automatic fallback); added fuel type
and ignition method controls to the smoke screening cone; fixed a print bug that could
cut off the LVORI column on the 7-day forecast printout; added pin-drop location with
live NWS fire weather, the full hourly + 7-day tabular forecast, and print support for
both the smoke map and the 7-day table.
