# Accessing NamSoil Layers from Zenodo

Quick-start guide for reading, visualising, and cropping [NamSoil](https://zenodo.org/records/4090927) raster layers directly from Zenodo using R — no full download required.

## Requirements

- **R** ≥ 4.1
- **terra** package (`install.packages("terra")`)
- GDAL with `/vsicurl/` support (included with standard terra installations)

## Usage

```r
library(terra)

# Read layer on-the-fly from Zenodo (no download needed)
NamSoil_layer <- "https://zenodo.org/records/4090927/files/sol_log.oc_m_30m_0..20cm_2001..2017_v0.13_wgs84.tif"
Nam_layer <- terra::rast(NamSoil_layer, vsi = TRUE)

# Quick visualisation
terra::plot(Nam_layer)

# Define and crop to area of interest
ext <- terra::ext(17, 18, -20, -19)  # xmin, xmax, ymin, ymax
Nam_layer_cp <- terra::crop(Nam_layer[[1]], ext)

# Save cropped raster to disk
terra::writeRaster(Nam_layer_cp, "aoi_Nam_soil_layer.tif",
                   gdal = c("COMPRESS=NONE", "TFW=YES"), overwrite = TRUE)
```

## How it works

The script uses GDAL's [`/vsicurl/`](https://gdal.org/en/stable/user/virtual_file_systems.html#vsicurl-http-https-ftp-files-random-access) virtual file system, enabled via `vsi = TRUE` in `terra::rast()`. This reads raster data on-the-fly through HTTP range requests, so only the requested pixels are downloaded rather than the entire file.

## Customising the area of interest

The extent can be defined in two ways:

- **Bounding box:** `terra::ext(xmin, xmax, ymin, ymax)` as shown above.
- **Shapefile:** `terra::vect("your_shapefile.shp")` for an irregular boundary.

## Available bands

Each NamSoil GeoTIFF may contain multiple bands:

| Band | Description |
|------|-------------|
| 1 | Predicted mean |
| 2 | 95th percentile (upper limit) |
| 3 | 5th percentile (lower limit) |
| 4 | 90% prediction interval (PI₉₀) |

Use `Nam_layer[[1]]` to select a specific band, or omit the index to crop all bands at once.

## Notes

- The Zenodo URL in the example points to a specific layer; replace it with the URL of any other NamSoil product layer.
- Large areas or full-country reads over `/vsicurl/` can be slow — for heavy processing consider downloading the full file first.

## Reference

NamSoil spatial predictions are described in the associated Zenodo record: [https://zenodo.org/records/4090927](https://zenodo.org/records/4090927).
