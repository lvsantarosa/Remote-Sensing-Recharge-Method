# Remote Sensing Recharge Method

Remote sensing workflow to quantify **potential aquifer recharge (PR)** and to compare it with
**groundwater storage (GWS)** variation, as applied in:

> Santarosa, L. V., Pinto, G. V. F., Blandón Luengas, J. S., & Gastmans, D. (2024).
> *Remote sensing to quantify potential aquifer recharge as a complementary tool for groundwater monitoring.*
> **Hydrological Sciences Journal**. https://doi.org/10.1080/02626667.2024.2412741

This repository merges two previously separate code bases into a single, end-to-end pipeline:

| Module | What it does | Previous repository |
|---|---|---|
| [`01_PR_METHOD/`](01_PR_METHOD) | Potential recharge from a distributed water budget: `PR = P - (ETP + Roff)`, with runoff from the SCS-CN method | [Aquifer-recharge-with-PR-method](https://github.com/lvsantarosa/Aquifer-recharge-with-PR-method) |
| [`02_GWS_GRACE/`](02_GWS_GRACE) | Groundwater storage variation and annual maximum recharge per pixel, used as the independent reference for the PR estimates | [Aquifer-recharge-with-GWS-data](https://github.com/lvsantarosa/Aquifer-recharge-with-GWS-data) |

## Workflow

```
                 ┌──────────────── 01_PR_METHOD ────────────────┐
 SoilGrids (250 m) ─┐
                    ├─► Curve Number (CN) ─┐
 MapBiomas LULC ────┘                      │
                                           ├─► Daily runoff (SCS-CN)
 GPM precipitation ────────────────────────┘            │
                                                        ▼
 GLDAS 2.1 evapotranspiration ─────────────► PR = P - (ETP + Roff)
                                                        │  monthly → annual
                 ┌──────────────── 02_GWS_GRACE ────────┼───────┐
 GLDAS 2.2 CLSM (GRACE DA) GWS_tavg ─► daily GWS ─► significant
   recharge/depletion events ─► annual maximum recharge per pixel
                                                        │
                                                        ▼
                                         comparison PR vs. GWS recharge
```

## Repository structure

```
01_PR_METHOD/
  00_SCS_TO_PR_METHOD.R            # soil texture → hydrologic group, LULC reclass, Curve Number
  01_PR_AQUIFER_RECHARGE_METHOD.R  # daily runoff, monthly water budget, annual recharge rasters
02_GWS_GRACE/
  00_DOWNLOAD_GWS_GEE.ipynb        # GWS download from Google Earth Engine (Colab)
  01_GWS_PROCESS_AND_RECHARGE.R    # GWS time series per grid cell → annual maximum recharge
  GRID.gpkg                        # analysis grid (one polygon per GWS pixel)
  SAB_GWS.rar                      # example GWS data (Guarani Aquifer System outcrop area)
```

The two modules are independent: each can be run on its own, and they only meet at the
comparison stage, where the annual PR rasters and the annual GWS recharge rasters share the
same study area and hydrological years.

## Input data

| Variable | Product | Source |
|---|---|---|
| Precipitation (P) | GPM, daily | NASA GES DISC — [download instructions](https://github.com/lvsantarosa/NASA-data-download) |
| Evapotranspiration (ETP) | GLDAS 2.1, monthly | NASA GES DISC — [download instructions](https://github.com/lvsantarosa/NASA-data-download) |
| Groundwater storage (GWS) | GLDAS 2.2 CLSM with GRACE data assimilation, `GWS_tavg`, daily | Google Earth Engine — [`02_GWS_GRACE/00_DOWNLOAD_GWS_GEE.ipynb`](02_GWS_GRACE/00_DOWNLOAD_GWS_GEE.ipynb) and [NASA-data-download](https://github.com/lvsantarosa/NASA-data-download) |
| Soil texture (clay/sand/silt) | SoilGrids, 250 m | [Soil-Grid-on-Google-Earth-Engine](https://github.com/lvsantarosa/Soil-Grid-on-Google-Earth-Engine) |
| Land use / land cover | MapBiomas (Brazil) | [mapbiomas.org](https://mapbiomas.org/) |

A worked example with processed data for 2023 is available
[here](https://drive.google.com/drive/folders/1gQTgBNakUZKlkHiiFL14R_Fw8En824Qe?usp=drive_link).

## Requirements

R (≥ 4.2) with `terra`, `sf`, `ncdf4`, `tidyverse` (`dplyr`, `purrr`, `tidyr`), `lubridate`,
`glue` and `soiltexture`:

```r
install.packages(c("terra", "sf", "ncdf4", "tidyverse", "lubridate", "glue", "soiltexture"))
```

The notebook in `02_GWS_GRACE/` runs on Google Colab and needs `earthengine-api`, `geemap`
and `geopandas`, plus a Google Earth Engine account.

## How to run

1. **Curve Number** — run `01_PR_METHOD/00_SCS_TO_PR_METHOD.R` with the SoilGrids and MapBiomas
   rasters in place. It writes `CN_<year>.tif` and the hydrologic soil group raster.
2. **Potential recharge** — run `01_PR_METHOD/01_PR_AQUIFER_RECHARGE_METHOD.R`. It resamples the
   CN, computes daily runoff, aggregates precipitation and runoff to monthly totals, applies
   `PR = P - (ETP + Roff)` and aggregates to annual recharge rasters.
3. **GWS recharge** — run `02_GWS_GRACE/00_DOWNLOAD_GWS_GEE.ipynb` to download the GWS series,
   then `02_GWS_GRACE/01_GWS_PROCESS_AND_RECHARGE.R` to extract, per grid cell, the daily level
   variation, the significant recharge events and the annual maximum recharge, exported as a
   shapefile and as one raster per hydrological year.

Each script starts with a `setwd()` and expects the folder layout described in the module
READMEs — adjust those paths to your own machine before running.

## Citation

If you use this code, please cite the paper above. Each module also documents the specific
method it implements in its own README.
