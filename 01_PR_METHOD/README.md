# 01 — Potential recharge (PR method)

Potential aquifer recharge from a distributed monthly water budget:

```
PR = P - (ETP + Roff)
```

where `P` is precipitation (GPM), `ETP` is evapotranspiration (GLDAS 2.1) and `Roff` is surface
runoff computed with the SCS Curve Number method.

## Scripts

### `00_SCS_TO_PR_METHOD.R` — Curve Number

1. Reads SoilGrids clay/sand/silt (250 m) and classifies each pixel into an SCS hydrologic soil
   group (A = 1, B = 2, C = 3, D = 4) through the USDA textural triangle.
2. Reads MapBiomas land cover and reclassifies it into SCS land-use classes
   (urban, agriculture, forest, silviculture, grassland, bare soil, pasture, water).
3. Adds the soil and land-use codes and reclassifies the sum into CN values, following the
   USDA's *Urban Hydrology for Small Watersheds*
   ([TR-55](https://www.nrcs.usda.gov/Internet/FSE_DOCUMENTS/stelprdb1044171.pdf)).
4. Exports `CN_<year>.tif` and the hydrologic soil group raster.

Class codes differ between MapBiomas collections — check `class_map_scs` against the collection
you downloaded before running.

### `01_PR_AQUIFER_RECHARGE_METHOD.R` — runoff and recharge

1. Reprojects and resamples the daily GPM rasters to the analysis grid, and aggregates them to
   monthly totals.
2. Resamples the CN raster to the same grid.
3. Computes daily runoff: `S = 25400/CN - 254`, `Ia = 0.2 S`,
   `Roff = (P - Ia)² / (P - Ia + S)`, with `Roff = 0` where `Ia > P`; then aggregates to monthly.
4. Converts GLDAS evapotranspiration from rate to monthly depth, applies the water budget and
   writes monthly and annual recharge rasters.

## Expected folder layout

```
01_DADOS/
  00_SOIL/              clay.tif, sand.tif, silt.tif (SoilGrids)
  01_LULC/              MapBiomas coverage rasters (.tif)
  02_PRECIPITATION/     GPM daily files (.nc4)
  03_EVAPOTRANSPIRATION/ GLDAS 2.1 monthly files (.SUB)
02_RESULTADOS/
  00_CN/                CN_<year>.tif, 00_SOIL_HIDRO/, 01_CN_RES/
  01_RUNOFF_DAY/  02_RUNOFF_MONTH/
  03_PRECIPITATION/  05_PRECIPITATION_MONTH/
  04_EVAPOTRANPIRATION/
  06_RECHARGE_MONTH/  07_RECHARGE_YEAR/
```

Set the study area in the `grid` object (extent and resolution) and the period in the `st`/`en`
dates — both scripts must use the same grid.
