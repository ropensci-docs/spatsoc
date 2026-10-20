# Build polygons

`build_polys` generates a simple feature collection with POLYGONs from a
`data.table`. The function expects a `data.table` with relocation data,
individual identifiers, a crs, home range type and parameters. The
relocation data is transformed into POLYGONs using either
[adehabitatHR::mcp](https://rdrr.io/pkg/adehabitatHR/man/mcp.html) or
[adehabitatHR::kernelUD](https://rdrr.io/pkg/adehabitatHR/man/kernelUD.html)
for each individual and, optionally, combination of columns listed in
`splitBy`. Relocation data should be in two columns representing the X
and Y coordinates.

## Usage

``` r
build_polys(
  DT = NULL,
  crs = NULL,
  hrType = NULL,
  hrParams = NULL,
  id = NULL,
  coords = NULL,
  splitBy = NULL,
  spPts = NULL,
  projection = NULL
)
```

## Arguments

- DT:

  input data.table

- crs:

  numeric or character defining the coordinate reference system to be
  passed to
  [sf::st_crs](https://r-spatial.github.io/sf/reference/st_crs.html).
  For example, either `crs = "EPSG:32736"` or `crs = 32736`.

- hrType:

  type of HR estimation, either 'mcp' or 'kernel'

- hrParams:

  a named list of parameters for `adehabitatHR` functions

- id:

  character string of ID column name

- coords:

  character vector of X coordinate and Y coordinate column names. Note:
  the order is assumed X followed by Y column names

- splitBy:

  (optional) character string or vector of grouping column name(s) upon
  which the output will be calculated

- spPts:

  alternatively, provide solely a SpatialPointsDataFrame with one column
  representing the ID of each point, as specified by
  [adehabitatHR::mcp](https://rdrr.io/pkg/adehabitatHR/man/mcp.html) or
  [adehabitatHR::kernelUD](https://rdrr.io/pkg/adehabitatHR/man/kernelUD.html)

- projection:

  (deprecated) use crs argument instead

## Value

`build_polys` returns a simple feature collection with POLYGONs for each
individual (and optionally `splitBy` combination).

An error is returned when `hrParams` do not match the arguments of the
respective `hrType` `adehabitatHR` function.

## Details

[group_polys](https://docs.ropensci.org/spatsoc/reference/group_polys.md)
uses `build_polys` for grouping overlapping polygons created from
relocations.

### R-spatial evolution

Please note, spatsoc has followed updates from R spatial, GDAL and PROJ
for handling coordinate reference systems, see more below and details at
<https://r-spatial.org/r/2020/03/17/wkt.html>.

In addition, `build_polys` previously used
[sp::SpatialPoints](https://edzer.github.io/sp/reference/SpatialPoints.html)
but has been updated to use
[sf::st_as_sf](https://r-spatial.github.io/sf/reference/st_as_sf.html)
according to the R-spatial evolution, see more at
<https://r-spatial.org/r/2022/04/12/evolution.html>.

### Notes on arguments

The `DT` must be a `data.table`. If your data is a `data.frame`, you can
convert it by reference using
[data.table::setDT](https://rdrr.io/pkg/data.table/man/setDT.html).

The `id`, `coords` (and optional `splitBy`) arguments expect the names
of respective columns in `DT` which correspond to the individual
identifier, X and Y coordinates, and additional grouping columns.

The `crs` argument expects a character string or numeric defining the
coordinate reference system to be passed to
[sf::st_crs](https://r-spatial.github.io/sf/reference/st_crs.html). For
example, for UTM zone 36S (EPSG 32736), the crs argument is
`crs = "EPSG:32736"` or `crs = 32736`. See
<https://spatialreference.org> for a list of EPSG codes.

The `hrType` must be either one of "kernel" or "mcp". The `hrParams`
must be a named list of arguments matching those of
[adehabitatHR::kernelUD](https://rdrr.io/pkg/adehabitatHR/man/kernelUD.html)
and
[adehabitatHR::getverticeshr](https://rdrr.io/pkg/adehabitatHR/man/getverticeshr.html)
or [adehabitatHR::mcp](https://rdrr.io/pkg/adehabitatHR/man/mcp.html).

The `splitBy` argument offers further control building POLYGONs. If in
your `DT`, you have multiple temporal groups (e.g.: years) for example,
you can provide the name of the column which identifies them and build
POLYGONs for each individual in each year.

## See also

[group_polys](https://docs.ropensci.org/spatsoc/reference/group_polys.md)

Other Build functions:
[`build_lines()`](https://docs.ropensci.org/spatsoc/reference/build_lines.md)

## Examples

``` r
# Load data.table
library(data.table)

# Read example data
DT <- fread(system.file("extdata", "DT.csv", package = "spatsoc"))

# Cast the character column to POSIXct
DT[, datetime := as.POSIXct(datetime, tz = 'UTC')]
#>            ID        X       Y            datetime population
#>        <char>    <num>   <num>              <POSc>      <int>
#>     1:      A 715851.4 5505340 2016-11-01 00:00:54          1
#>     2:      A 715822.8 5505289 2016-11-01 02:01:22          1
#>     3:      A 715872.9 5505252 2016-11-01 04:01:24          1
#>     4:      A 715820.5 5505231 2016-11-01 06:01:05          1
#>     5:      A 715830.6 5505227 2016-11-01 08:01:11          1
#>    ---                                                       
#> 14293:      J 700616.5 5509069 2017-02-28 14:00:54          1
#> 14294:      J 700622.6 5509065 2017-02-28 16:00:11          1
#> 14295:      J 700657.5 5509277 2017-02-28 18:00:55          1
#> 14296:      J 700610.3 5509269 2017-02-28 20:00:48          1
#> 14297:      J 700744.0 5508782 2017-02-28 22:00:39          1

# EPSG code for example data
utm <- 32736

# Build polygons for each individual using kernelUD and getverticeshr
build_polys(DT, crs = utm, hrType = 'kernel',
            hrParams = list(grid = 60, percent = 95),
            id = 'ID', coords = c('X', 'Y'))
#> Simple feature collection with 10 features and 2 fields
#> Geometry type: MULTIPOLYGON
#> Dimension:     XY
#> Bounding box:  xmin: 691964.9 ymin: 5489527 xmax: 717689.4 ymax: 5516125
#> Projected CRS: +proj=utm +zone=36 +south +datum=WGS84 +units=m +no_defs
#>   ID      area                       geometry
#> A  A 192345367 MULTIPOLYGON (((693628 5506...
#> B  B  17316394 MULTIPOLYGON (((703312.2 55...
#> C  C 151238156 MULTIPOLYGON (((694712.6 55...
#> D  D  16127640 MULTIPOLYGON (((694883.4 54...
#> E  E  75190273 MULTIPOLYGON (((694705.4 55...
#> F  F 115041937 MULTIPOLYGON (((694898.6 55...
#> G  G  23395212 MULTIPOLYGON (((702963.8 55...
#> H  H 101217169 MULTIPOLYGON (((692020.5 55...
#> I  I 176667205 MULTIPOLYGON (((694732 5505...
#> J  J 192475536 MULTIPOLYGON (((693703 5506...

# Build polygons for each individual by year
DT[, yr := year(datetime)]
#>            ID        X       Y            datetime population    yr
#>        <char>    <num>   <num>              <POSc>      <int> <int>
#>     1:      A 715851.4 5505340 2016-11-01 00:00:54          1  2016
#>     2:      A 715822.8 5505289 2016-11-01 02:01:22          1  2016
#>     3:      A 715872.9 5505252 2016-11-01 04:01:24          1  2016
#>     4:      A 715820.5 5505231 2016-11-01 06:01:05          1  2016
#>     5:      A 715830.6 5505227 2016-11-01 08:01:11          1  2016
#>    ---                                                             
#> 14293:      J 700616.5 5509069 2017-02-28 14:00:54          1  2017
#> 14294:      J 700622.6 5509065 2017-02-28 16:00:11          1  2017
#> 14295:      J 700657.5 5509277 2017-02-28 18:00:55          1  2017
#> 14296:      J 700610.3 5509269 2017-02-28 20:00:48          1  2017
#> 14297:      J 700744.0 5508782 2017-02-28 22:00:39          1  2017
build_polys(DT, crs = utm, hrType = 'mcp',
            hrParams = list(percent = 95),
            id = 'ID', coords = c('X', 'Y'), splitBy = 'yr')
#> Simple feature collection with 20 features and 2 fields
#> Geometry type: POLYGON
#> Dimension:     XY
#> Bounding box:  xmin: 693394.8 ymin: 5490131 xmax: 715229.7 ymax: 5514806
#> Projected CRS: +proj=utm +zone=36 +south +datum=WGS84 +units=m +no_defs
#> First 10 features:
#>            ID      area                       geometry
#> A-2016 A-2016 141208128 POLYGON ((707624.4 5514064,...
#> A-2017 A-2017  68039437 POLYGON ((706582.7 5510383,...
#> B-2016 B-2016   7851014 POLYGON ((699361.8 5511721,...
#> B-2017 B-2017   9473597 POLYGON ((698473.2 5512210,...
#> C-2016 C-2016  90247899 POLYGON ((711698.9 5506990,...
#> C-2017 C-2017  64419958 POLYGON ((706580.1 5510401,...
#> D-2016 D-2016   9478060 POLYGON ((698487.1 5492921,...
#> D-2017 D-2017   7624492 POLYGON ((699711.3 5492069,...
#> E-2016 E-2016  60086909 POLYGON ((707874.5 5507416,...
#> E-2017 E-2017  23034604 POLYGON ((700660.7 5508942,...
```
