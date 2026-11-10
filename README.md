# Drone vs PNOA LiDAR viewer — Santomera

Static web viewer for the cell-by-cell comparison of wildfire-fuel metrics
between a high-density drone LiDAR flight (2026, used as ground truth) and the
Spanish national LiDAR coverage (PNOA 2023), on a common 10 m grid. It shows
the learned PNOA→drone calibration, residuals before/after calibration and the
cells that fall outside the 90% conformal prediction interval.

## Run

```bash
cd visor
python3 -m http.server 8123
# open http://localhost:8123/
```

(A local HTTP server is needed; opening `index.html` via `file://` fails on
the data `fetch`.)

## What it shows

- **Layer selector** with six views: drone (ground truth), PNOA raw, PNOA
  calibrated, raw residual, calibrated residual and cells outside the 90%
  conformal interval. All sources share the same colour scale, so switching
  layers is a fair comparison.
- **Four fuel metrics**: canopy height p95, canopy cover (> 2 m), understorey
  (1–5 m) and mid-storey (5–15 m).
- **Change 2023–26 overlay** (orange): pine mortality/felling between the 2023
  PNOA flight and the 2026 drone flight (12.4 ha, confirmed with 2022–2026
  orthophotos). Real change, not measurement error; excluded from calibration.
- **Click any cell**: drone / PNOA / calibrated values and whether the
  prediction interval covers the drone value.
- Top-right panel: MAE before/after calibration and empirical interval coverage.
- Residual = PNOA − drone; red = overestimation, blue = underestimation.
- The in-app **? Help** button explains every layer and convention in detail.

## Data

`datos/rejilla.json` is a precomputed export of the analysis pipeline
(10 m grid: drone and PNOA fuel metrics, calibrated predictions, conformal
quantiles and the 2023–2026 change mask). To update the site, just replace
this file with a newer export.

The interval shown is the **global** split-conformal 90% variant over
out-of-fold residuals of the gbt model.

## Deploy (any GitHub-connected static host)

The viewer is a fully static site — **no build step**:

- **Build command:** *(none)*
- **Publish / output directory:** *(repository root)*
- **No environment variables, no serverless functions.**

Everything the page needs lives in this repo (`index.html`, `datos/`,
`vendor/` with Leaflet 1.9.4 and proj4js 2.11.0 vendored), and all internal
paths are relative, so the site works deployed at the domain root or at a
subpath (e.g. `https://host.example/mi-repo/`). The only outbound requests at
runtime are the basemap tiles (IGN WMTS orthophoto and CARTO, both plain
public HTTPS tile endpoints).

## Basemap & stack

IGN PNOA orthophoto (WMTS); dark alternative from CARTO/OSM.
Leaflet 1.9.4 + proj4js (ETRS89 UTM30 → WGS84), vendored in `vendor/`,
no build step.
