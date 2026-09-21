# Get geometry

`get_geometry` sets up an input `DT` with a 'geometry' column for
spatsoc's geometry interface. The function expects a `data.table` with
relocation data and a coordinate reference system.

## Usage

``` r
get_geometry(
  DT = NULL,
  coords = NULL,
  crs = NULL,
  output_crs = NULL,
  geometry_colname = "geometry"
)
```

## Arguments

- DT:

  input data.table

- coords:

  character vector of X coordinate and Y coordinate column names. Note:
  the order is assumed X followed by Y column names

- crs:

  numeric or character defining the coordinate reference system to be
  passed to
  [sf::st_crs](https://r-spatial.github.io/sf/reference/st_crs.html).
  For example, `crs = "EPSG:32736"` or `crs = 32736`.

- output_crs:

  default NULL, the output crs to transform the input coordinates to
  with
  [sf::st_transform](https://r-spatial.github.io/sf/reference/st_transform.html).
  If output_crs is NULL or matching the crs argument, the coordinates
  will not be transformed

- geometry_colname:

  default "geometry", to optionally set output name of simple feature
  geometry list column

## Value

`get_geometry` returns the input `DT` appended with a `geometry` column
which represents the input coordinates as a `sfc` (simple feature
geometry list column). If the `output_crs` was provided, the `geometry`
will be transformed to the `output_crs`.

A message is returned when a column named `geometry` already exists in
the input `DT`, because it will be overwritten.

See details for appending outputs using modify-by-reference in the
[FAQ](https://docs.ropensci.org/spatsoc/articles/faq.html).

## Details

The `DT` must be a `data.table`. If your data is a `data.frame`, you can
convert it by reference using
[`data.table::setDT()`](https://rdrr.io/pkg/data.table/man/setDT.html)
or by reassigning using
[`data.table::data.table()`](https://rdrr.io/pkg/data.table/man/data.table.html).

The `coords` argument expects the names of columns in `DT` which
correspond to the X and Y coordinates.

The `output_crs` argument allows the user to set an output `crs` for
their geometry column. Note: some functions in spatsoc (eg. those that
measure directions like `edge_direction` and `direction_to_leader`)
require geographic coordinates and it is therefore simpler to leave the
default `output_crs = 4326`.

## Examples

``` r
# Load data.table
library(data.table)

# Read example data
DT <- fread(system.file('extdata', 'DT.csv', package = 'spatsoc'))

# Get geometry
get_geometry(DT, coords = c('X', 'Y'), crs = 32736)
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
#>                        geometry
#>                     <sfc_POINT>
#>     1: POINT (715851.4 5505340)
#>     2: POINT (715822.8 5505289)
#>     3: POINT (715872.9 5505252)
#>     4: POINT (715820.5 5505231)
#>     5: POINT (715830.6 5505227)
#>    ---                         
#> 14293: POINT (700616.5 5509069)
#> 14294: POINT (700622.6 5509065)
#> 14295: POINT (700657.5 5509277)
#> 14296: POINT (700610.3 5509269)
#> 14297:   POINT (700744 5508782)

# Print
print(DT)
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
#>                        geometry
#>                     <sfc_POINT>
#>     1: POINT (715851.4 5505340)
#>     2: POINT (715822.8 5505289)
#>     3: POINT (715872.9 5505252)
#>     4: POINT (715820.5 5505231)
#>     5: POINT (715830.6 5505227)
#>    ---                         
#> 14293: POINT (700616.5 5509069)
#> 14294: POINT (700622.6 5509065)
#> 14295: POINT (700657.5 5509277)
#> 14296: POINT (700610.3 5509269)
#> 14297:   POINT (700744 5508782)
```
