# marine-forecast

Publishes [weather.brianbeals.com](https://weather.brianbeals.com), a branded
marine forecast for Charlotte Harbor and Pine Island Sound (Burnt Store to
Boca Grande Pass).

## How it works

`build.py` fetches the NOAA/NWS/FWC sources below and renders a single static
`index.html`:

| Source | What | Required |
|---|---|---|
| api.weather.gov products (CWFTBW) | Wind, chop, Gulf seas, advisories | Yes. Stale/missing product aborts; last good page stays up |
| CO-OPS predictions, Port Boca Grande 8725577 | Tides + fishing windows | No |
| api.weather.gov gridpoints TBW/91,55 forecast | Air temps, sky, rain % per period | No |
| api.weather.gov gridpoints TBW/91,55 (raw) | Hourly wind, gust, rain % for the timeline | No |
| api.weather.gov gridpoints TBW/77,42 (raw) | Modeled nearshore Gulf wave height + period | No |
| NHC Atlantic Tropical Weather Outlook | Tropics line | No |
| FWC HAB current status (ArcGIS feature layer) | Red tide sample status near the harbor | No |
| CO-OPS latest obs, Fort Myers 8725520 | Live wind + water temp | No |

The page leads with a condition-based verdict line (chop, storm risk, and rain
scored into a plain go / caution / marginal call), then situational lines: the
NHC tropics outlook and a five-step red-tide stoplight (green none to red high,
linked to the FWC map). The Today hero shows wind, water state and storm risk as
color pills, rain, air, tides and best bite, plus the CWF Gulf seas and a modeled
nearshore wave reading (current + today's peak) for the pass run.

A "Wind & gusts today" card charts the hourly sustained wind and gusts, marks the
peak, shades each hour green / yellow / red for go / marginal / don't-go, and
names the cleanest window, all from the raw gridpoint (gusts + rain). A live
"now" line tracks the current hour across it.

A Tides & Best Fishing section carries a per-day tide curve (cosine-interpolated
between the highs and lows so it takes the real tide shape) with the best-bite
windows shaded and dawn/dusk marked, next to the Port Boca Grande hi/lo times
and a tide-plus-light bite rating.

### Live in the browser (on load, every 5 min)

- Right-now observations (CO-OPS wind and water, KPGD sky, plus feels-like and
  visibility when they cross a threshold worth showing).
- Official NWS active alerts for the marine zones (GMZ836 harbor, GMZ856 Gulf)
  from `api.weather.gov/alerts/active`. Special Marine and Severe Thunderstorm
  warnings render red, Small Craft Advisories amber, at the top. Authoritative
  for warnings issued between builds; supersedes the baked CWF headline.
- The harbor-cropped NEXRAD loop over an Esri basemap (World Street by day, Dark
  Gray by night) with the coastline and labels layered above the radar so
  geography survives a cell.
- NEXRAD reflectivity sampled at several harbor points (Boca Grande Pass,
  Charlotte Harbor, Pine Island Sound, Burnt Store), classified by the n0q color
  ramp; posts a live storm ribbon when a cell is on the water, an early backstop
  before an official warning (steps aside when one is up). Canvas readback works
  because IEM serves CORS headers.

## Deploy

GitHub Actions (`.github/workflows/publish.yml`) polls every 10 minutes through
the hour after each expected NWS issuance (about 3:15 and 9:15, AM and PM
Eastern), rebuilds when the issuance is new, commits `index.html`, and deploys
to Pages through the Actions pipeline (`upload-pages-artifact` + `deploy-pages`).
A workflow-level concurrency group serializes runs so deploys never collide.
Pages source is set to "GitHub Actions."

## Source of truth

This repo is canonical. The `scripts/` here are what runs. The project began
as the Cowork `marine-forecast` skill in `brianbeals/claude-skills`, but has
since migrated to git; that skill copy is legacy and is no longer synced from.
Edit the scripts here.

## Local run

```bash
python3 build.py   # writes fetched/ + index.html
open index.html
```

DST is handled automatically (`zoneinfo`, America/New_York). Stdlib only, no
requirements file.
