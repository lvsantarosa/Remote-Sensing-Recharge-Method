# 02 — Groundwater storage (GWS) recharge

Groundwater recharge inferred from groundwater storage variation, used in the paper as the
independent reference against which the PR estimates are compared.

The GWS series comes from the GLDAS 2.2 CLSM product with GRACE data assimilation
(`NASA/GLDAS/V022/CLSM/G025/DA1D`, band `GWS_tavg`, daily, 0.25°).

## Scripts

### `00_DOWNLOAD_GWS_GEE.ipynb` — download (Google Colab)

Authenticates Google Earth Engine, defines the area of interest, computes the daily mean of
`GWS_tavg` and exports the images to Google Drive. Alternative download routes are documented in
[NASA-data-download](https://github.com/lvsantarosa/NASA-data-download).

### `01_GWS_PROCESS_AND_RECHARGE.R` — processing and recharge

For each cell of `GRID.gpkg`:

1. Builds the daily GWS time series (mean of the masked pixels).
2. Computes the day-to-day level variation.
3. Keeps only variations above a significance threshold (`threshold <- 0.5`).
4. Labels each day as a positive, negative or neutral period.
5. For every positive period, measures the rise from the last preceding negative level.
6. Takes the maximum rise per hydrological year (October–September) as that year's recharge.

Results are written as one CSV per cell, merged into a shapefile with one column per hydrological
year, and rasterized to one GeoTIFF per year.

## Files

- `GRID.gpkg` — analysis grid, one polygon per GWS pixel, with an `ID` field used to name outputs.
- `SAB_GWS.rar` — example GWS data for the Guarani Aquifer System (SAB) outcrop area, 2003-10-01
  to 2021-09-30.

## Expected folder layout

```
00_SHP/00_LIMITE_AREA/GRID.gpkg
01_GWS/01_DADOS/SAB_GWS/     GWS files (.nc4)
01_GWS/01_DADOS/CSV/         per-cell annual maxima (output)
02_RESULTADOS/               shapefile and annual rasters (output)
```

Adjust the date sequence (`start_date`/`end_date`), the `threshold` and the year range in the
rasterization loop to your own series.
