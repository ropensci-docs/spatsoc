# Build lines

`build_lines` generates a simple feature collection with LINESTRINGs
from a `data.table`. The function expects a `data.table` with relocation
data, individual identifiers, a sorting column and a `crs`. The
relocation data is transformed into LINESTRINGs for each individual and,
optionally, combination of columns listed in `splitBy`. Relocation data
should be in two columns representing the X and Y coordinates.

## Usage

``` r
build_lines(
  DT = NULL,
  crs = NULL,
  id = NULL,
  coords = NULL,
  sortBy = NULL,
  splitBy = NULL,
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
  For example, either `crs = "EPSG:32736"` or `crs = 32736`. Used only
  if coords are provided, see details under Interface

- id:

  character string of ID column name

- coords:

  character vector of X coordinate and Y coordinate column names. Note:
  the order is assumed X followed by Y column names

- sortBy:

  Character string of date time column(s) to sort rows by. Must be a
  POSIXct.

- splitBy:

  (optional) character string or vector of grouping column name(s) upon
  which the output will be calculated

- projection:

  (deprecated) use crs argument instead

## Value

`build_lines` returns an sf LINESTRING object with a line for each
individual (and optionally `splitBy` combination).

Individuals (or combinations of individuals and `splitBy`) with less
than two relocations are dropped since it requires at least two
relocations to build a line.

## Details

### R-spatial evolution

Please note, spatsoc has followed updates from R spatial, GDAL and PROJ
for handling coordinate reference systems, see more at
<https://r-spatial.org/r/2020/03/17/wkt.html>.

In addition, `build_lines` previously used
[sp::SpatialLines](https://edzer.github.io/sp/reference/SpatialLines.html)
but has been updated to use
[sf::st_as_sf](https://r-spatial.github.io/sf/reference/st_as_sf.html)
and
[sf::st_linestring](https://r-spatial.github.io/sf/reference/st.html)
according to the R-spatial evolution, see more at
<https://r-spatial.org/r/2022/04/12/evolution.html>.

### Notes on arguments

The `crs` argument expects a numeric or character defining the
coordinate reference system. For example, for UTM zone 36N (EPSG 32736),
the crs argument is either `crs = 'EPSG:32736'` or `crs = 32736`. See
details in
[`sf::st_crs()`](https://r-spatial.github.io/sf/reference/st_crs.html)
and <https://spatialreference.org> for a list of EPSG codes.

The `sortBy` argument is used to order the input `DT` when creating sf
LINESTRINGs. It must a column in the input `DT` of type POSIXct to
ensure the rows are sorted by date time.

The `splitBy` argument offers further control building LINESTRINGs. If
in your input `DT`, you have multiple temporal groups (e.g.: years) for
example, you can provide the name of the column which identifies them
and build LINESTRINGs for each individual in each year.

`build_lines` is used by `group_lines` for grouping overlapping lines
generated from relocations.

## See also

`group_lines`

Other Build functions:
[`build_polys()`](https://docs.ropensci.org/spatsoc/reference/build_polys.md)

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

# Build lines for each individual
lines <- build_lines(DT, crs = utm, id = 'ID', coords = c('X', 'Y'),
            sortBy = 'datetime')

# Build lines for each individual by year
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
lines <- build_lines(DT, crs = utm, id = 'ID', coords = c('X', 'Y'),
            sortBy = 'datetime', splitBy = 'yr')
```
