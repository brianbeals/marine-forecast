# marine-forecast

Publishes [weather.brianbeals.com](https://weather.brianbeals.com), a branded
marine forecast for Charlotte Harbor and Pine Island Sound (Burnt Store to
Boca Grande Pass).

## How it works

A build pipeline fetches the NOAA/NWS/FWC sources below and renders a single
static `index.html`:

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
and a tide-plus-light bite rating. Today's curve carries the same live "now"
line as the wind chart.

### Live in the browser (on load, every 5 min)

- Right-now observations (CO-OPS wind and water, KPGD sky, plus feels-like and
  visibility when they cross a threshold worth showing).
- Official NWS active alerts for the marine zones (GMZ836 harbor, GMZ856 Gulf)
  from `api.weather.gov/alerts/active`. Special Marine and Severe Thunderstorm
  warnings render red, Small Craft Advisories amber, at the top. Each alert links
  to the affected marine-zone forecast page, which keeps showing the warning text
  after the short-lived alert itself expires. Authoritative for warnings issued
  between builds; supersedes the baked CWF headline.
- The harbor-cropped NEXRAD loop over an Esri basemap (World Street by day, Dark
  Gray by night) with the coastline and labels layered above the radar so
  geography survives a cell.
- NEXRAD reflectivity sampled at several harbor points (Boca Grande Pass,
  Charlotte Harbor, Pine Island Sound, Burnt Store), classified by the n0q color
  ramp, then gated against MRMS 2-minute surface accumulation (`mrms_a2m`) at the
  same points: the ribbon fires only where rain is actually reaching the ground.
  KTBW sits ~70 nm off, so its 0.5° beam is thousands of feet over the harbor and
  raw reflectivity flags virga and clear-air returns that never wet the water; the
  surface-precip gate drops those false fires while n0q still sets the intensity
  wording. A no-coverage a2m pixel holds the n0q read rather than falsely clearing
  it. Posts a live storm ribbon as an early backstop before an official warning
  (steps aside when one is up). MRMS is served from the same IEM host as the radar
  loop, so canvas readback needs no extra CORS.

## Deploy

The forecast is rendered by a build pipeline that fetches the sources above,
builds `index.html`, and pushes the finished page into this repo. This repo
publishes it with GitHub Pages (deploy from `main` / root) on the custom domain
weather.brianbeals.com. The build runs on a schedule that polls each NWS
issuance window (about 3:15 and 9:15, AM and PM Eastern) and publishes only when
the issuance is new. DST is handled automatically (`zoneinfo`,
America/New_York).

## What's here

This repo holds the published site: the rendered `index.html`, the custom-domain
`CNAME`, and the page assets. The build code that fetches the data and renders
the page lives in a separate private repository, so the pipeline itself isn't
part of this repo.
