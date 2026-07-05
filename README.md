# marine-forecast

Publishes [weather.brianbeals.com](https://weather.brianbeals.com), a branded
marine forecast for Charlotte Harbor and Pine Island Sound (Burnt Store to
Boca Grande Pass).

## How it works

`build.py` fetches five NOAA/NWS sources and renders a single static
`index.html`:

| Source | What | Required |
|---|---|---|
| api.weather.gov products (CWFTBW) | Wind, chop, Gulf seas, advisories | Yes. Stale/missing product aborts; last good page stays up |
| CO-OPS predictions, Port Boca Grande 8725577 | Tides + fishing windows | No |
| api.weather.gov gridpoints TBW/91,55 | Air temps, sky, rain % | No |
| NHC Atlantic Tropical Weather Outlook | Tropics line | No |
| CO-OPS latest obs, Fort Myers 8725520 | Live wind + water temp | No |

GitHub Actions (`.github/workflows/publish.yml`) polls every 10 minutes
through the hour after each expected NWS issuance (about 3:15 and 9:15, AM
and PM Eastern). `build.py` exits without committing when the fetched
issuance is already published, so only a new product produces a commit.
GitHub Pages serves the repo root at the `CNAME` domain.

The rendered page carries a condition-based verdict line (chop, storm risk,
and rain scored into a plain go / caution / marginal call) and does its own
live work in the browser on load and every five minutes:

- Refreshes the "Right now" observations (CO-OPS wind and water, KPGD sky).
- Animates the harbor-cropped NEXRAD loop.
- Posts official NWS active alerts for the marine zones (GMZ836 harbor,
  GMZ856 Gulf) from `api.weather.gov/alerts/active`. Special Marine Warnings
  and Severe Thunderstorm warnings render red, Small Craft Advisories amber,
  at the top of the page. This is the authoritative feed for warnings issued
  between the 4x/day forecast builds; when it loads it supersedes the baked
  CWF advisory headline.
- Samples NEXRAD reflectivity at several harbor points (Boca Grande Pass,
  Charlotte Harbor, Pine Island Sound, Burnt Store) off the same IEM frames,
  classifies each by the n0q color ramp, and posts a live storm ribbon over
  the banner when a cell is on the water. This catches convection sitting on
  the harbor that the inland KPGD observation alone would miss, and acts as an
  early backstop before an official warning is issued (it steps aside when one
  is). Canvas readback works because IEM serves CORS headers.

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
