
<!-- README.md is generated from README.Rmd. Please edit that file -->

# terrainr: Retrieve Data from the USGS National Map and Transform it for 3D Landscape Visualizations <a href='https://mikemahoney218.github.io/terrainr/'><img src="man/figures/logo.png" align="right" height="138.5"/></a>

<!-- badges: start -->

[![License:
MIT](https://img.shields.io/badge/license-MIT-green)](https://choosealicense.com/licenses/mit/)
[![CRAN
status](https://www.r-pkg.org/badges/version/terrainr)](https://CRAN.R-project.org/package=terrainr)
[![Lifecycle:
maturing](https://img.shields.io/badge/lifecycle-maturing-blue.svg)](https://www.tidyverse.org/lifecycle/#maturing)
[![codecov](https://codecov.io/gh/mikemahoney218/terrainr/branch/master/graph/badge.svg)](https://codecov.io/gh/mikemahoney218/terrainr)
[![Project Status: Active – The project has reached a stable, usable
state and is being actively
developed.](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active)
[![R build
status](https://github.com/mikemahoney218/terrainr/workflows/R-CMD-check/badge.svg)](https://github.com/mikemahoney218/terrainr/actions)

<!-- badges: end -->

## Overview

terrainr makes it easy to identify your area of interest from point
data, retrieve geospatial data (including orthoimagery and DEMs) for
areas of interest within the United States from the [National Map family
of APIs](https://viewer.nationalmap.gov/services/), and then process
that data into larger, joined images or crop it into tiles that can be
imported into the Unity rendering engine.

At the absolute simplest level, terrainr provides a convenient and
consistent API to downloading data from the National Map.

``` r
library(terrainr)
simulated_data <-  data.frame(id = seq(1, 100, 1),
                              lat = runif(100, 44.04905, 44.17609), 
                              lng = runif(100, -74.01188, -73.83493))

bbox <- get_coord_bbox(lat = simulated_data$lat, lng = simulated_data$lng) 
output_tiles <- get_tiles(bbox = bbox,
                          services = c("elevation", "ortho"))
```

``` r
# output_tiles is now a list of two vectors pointing to the elevation and 
# orthoimagery tiles we just downloaded -- here we're displaying the first
# of the ortho tiles
raster::plot(raster::raster(output_tiles[[2]][[1]]))
```

<img src="man/figures/naip.png" width="100%" />

Once downloaded, these images are in standard GeoTIFF or PNG formats and
can be used as expected with other utilities:

``` r
raster::plot(raster::raster(output_tiles[[1]][[1]]))elev
```

<img src="man/figures/elevation.png" width="100%" />

Additionally, terrainr provides functions to transform these tiles into
RAW images ready to be imported into the Unity rendering engine,
allowing you to fly or walk through your downloaded data sets in 3D or
VR:

``` r
merged_dem <- tempfile(fileext = ".tif")
merged_ortho <- tempfile(fileext = ".tif")
# we can call these vectors by name instead of position, too
merge_rasters(output_tiles$`3DEPElevation`, 
              merged_dem, 
              output_tiles$USGSNAIPPlus, 
              merged_ortho)

mapply(function(x, y) raster_to_raw_tiles(input_file = x, 
                                          output_prefix = tempfile(), 
                                          side_length = 4097, 
                                          raw = y),
       c(merged_dem, merged_ortho),
       c(TRUE, FALSE))

# With about ten minutes of movie magic (loading the files into Unity), 
# we can turn that into:
```

<img src="man/figures/unity-lakeside.jpg" width="100%" />

terrainr also includes functionality to merge and crop the files you’ve
downloaded, and to resize your area of interest so you’re sure to
download exactly the area you want. Additionally, the more time
intensive processing steps can all be monitored via the
[progressr](https://github.com/HenrikBengtsson/progressr) package, so
you’ll be more confident that your computer is still churning along and
not just hung. For more information, check out [the introductory
vignette](https://mikemahoney218.github.io/terrainr/articles/overview.html)
and [the guide to importing your data into
Unity.](https://mikemahoney218.github.io/terrainr/articles/unity_instructions.html)

## Available Datasets

The following datasets can currently be downloaded using `get_tiles` or
`hit_national_map_api`:

  - [3DEPElevation](https://elevation.nationalmap.gov/arcgis/rest/services/3DEPElevation/ImageServer):
    The USGS 3D Elevation Program (3DEP) Bare Earth DEM.
  - [USGSNAIPPlus](https://services.nationalmap.gov/arcgis/rest/services/USGSNAIPPlus/MapServer):
    National Agriculture Imagery Program (NAIP) and high resolution
    orthoimagery (HRO).
  - [nhd](https://hydro.nationalmap.gov/arcgis/rest/services/nhd/MapServer):
    A comprehensive set of digital spatial data that encodes information
    about naturally occurring and constructed bodies of surface water
    (lakes, ponds, and reservoirs), paths through which water flows
    (canals, ditches, streams, and rivers), and related entities such as
    point features (springs, wells, stream gauges, and dams).
  - [govunits](https://carto.nationalmap.gov/arcgis/rest/services/govunits/MapServer):
    Major civil areas for the Nation, including States or Territories,
    counties (or equivalents), Federal and Native American areas,
    congressional districts, minor civil divisions, incorporated places
    (such as cities and towns), and unincorporated places.
  - [contours](https://carto.nationalmap.gov/arcgis/rest/services/contours/MapServer):
    The USGS Elevation Contours service.
  - [geonames](https://carto.nationalmap.gov/arcgis/rest/services/geonames/MapServer):
    Information about physical and cultural geographic features,
    geographic areas, and locational entities that are generally
    recognizable and locatable by name.
  - [NHDPlus\_HR](https://hydro.nationalmap.gov/arcgis/rest/services/NHDPlus_HR/MapServer):
    A comprehensive set of digital spatial data comprising a nationally
    seamless network of stream reaches, elevation-based catchment areas,
    flow surfaces, and value-added attributes.
  - [structures](https://carto.nationalmap.gov/arcgis/rest/services/structures/MapServer):
    The name, function, location, and other core information and
    characteristics of selected manmade facilities.
  - [transportation](https://carto.nationalmap.gov/arcgis/rest/services/transportation/MapServer):
    Roads, railroads, trails, airports, and other features associated
    with the transport of people or commerce.
  - [wbd](https://hydro.nationalmap.gov/arcgis/rest/services/wbd/MapServer):
    Hydrologic Unit (HU) polygon boundaries for the United States,
    Puerto Rico, and the U.S. Virgin Islands.

(All descriptions above taken from the [National Map API
descriptions](https://viewer.nationalmap.gov/services/).)

## Installation

You can install the development version of terrainr from
[GitHub](https://github.com/mikemahoney218/terrainr) with:

``` r
# install.packages("devtools")
devtools::install_github("mikemahoney218/terrainr")
```

## Code of Conduct

Please note that this package is released with a [Contributor Code of
Conduct](https://ropensci.org/code-of-conduct/). By contributing to this
project, you agree to abide by its terms.
