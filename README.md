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
GitHub Pages serves the repo root at the `CNAME` domain. The published page
also refreshes its "Right now" observations client-side on every load.

## Script sync

`scripts/render_forecast.py` and `scripts/marine_extras.py` are copies. The
canonical source is `brianbeals/claude-skills` →
`skills/marine-forecast/scripts/` (used interactively by the Cowork
`marine-forecast` skill). When the canonical scripts change, re-copy both
files here, keeping the `# SYNC:` header.

## Local run

```bash
python3 build.py   # writes fetched/ + index.html
open index.html
```

DST is handled automatically (`zoneinfo`, America/New_York). Stdlib only, no
requirements file.
