# Group polygons

`group_polys` groups rows into spatial groups by overlapping polygons
(home ranges). The function expects a `data.table` with relocation data,
individual identifiers and an `area` argument. The relocation data is
transformed into home range POLYGONs using
[`build_polys()`](https://docs.ropensci.org/spatsoc/reference/build_polys.md)
with [adehabitatHR::mcp](https://rdrr.io/pkg/adehabitatHR/man/mcp.html)
or
[adehabitatHR::kernelUD](https://rdrr.io/pkg/adehabitatHR/man/kernelUD.html).
If the `area` argument is `FALSE`, `group_polys` returns grouping
calculated by spatial overlap. If the `area` argument is `TRUE`,
`group_polys` returns the area area and proportion of overlap.
Relocation data should be in two columns representing the X and Y
coordinates.

## Usage

``` r
group_polys(
  DT = NULL,
  area = NULL,
  hrType = NULL,
  hrParams = NULL,
  crs = NULL,
  id = NULL,
  coords = NULL,
  splitBy = NULL,
  sfPolys = NULL,
  projection = NULL
)
```

## Arguments

- DT:

  input data.table

- area:

  logical indicating either overlap group (when `FALSE`) or area and
  proportion of overlap (when `TRUE`)

- hrType:

  type of HR estimation, either 'mcp' or 'kernel'

- hrParams:

  a named list of parameters for `adehabitatHR` functions

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

- splitBy:

  (optional) character string or vector of grouping column name(s) upon
  which the output will be calculated

- sfPolys:

  Alternatively, provide solely a simple features object with POLYGONs
  or MULTIPOLYGONs. If sfPolys are provided, id is required and splitBy
  cannot be used.

- projection:

  (deprecated) use crs argument instead

## Value

When `area` is `FALSE`, `group_polys` returns the input `DT` appended
with a `group` column. As with the other grouping functions, the actual
value of `group` is arbitrary and represents the identity of a given
group where 1 or more individuals are assigned to a group. If the data
was reordered, the `group` may change, but the contents of each group
would not. When `area` is `TRUE`, `group_polys` returns a proportional
area overlap `data.table`. In this case, ID refers to the focal
individual of which the total area is compared against the overlapping
area of ID2.

If `area` is `FALSE`, a message is returned when a column named `group`
already exists in the input `DT`, because it will be overwritten.

Along with changes to follow the R-spatial evolution, `group_polys` also
now returns area and proportion of overlap with units explicitly
specified through the `units` package.

Note: if area is TRUE, the output of `group_polys` needs to be
reassigned. See details in
[FAQ](https://docs.ropensci.org/spatsoc/articles/faq.html).

## Details

### R-spatial evolution

Please note, spatsoc has followed updates from R spatial, GDAL and PROJ
for handling coordinate reference systems, see more below and details at
<https://r-spatial.org/r/2020/03/17/wkt.html>.

In addition, `group_polys` previously used rgeos::gIntersection,
rgeos::gIntersects and rgeos::gArea but has been updated to use
[sf::st_intersects](https://r-spatial.github.io/sf/reference/geos_binary_pred.html),
[sf::st_intersection](https://r-spatial.github.io/sf/reference/geos_binary_ops.html)
and
[sf::st_area](https://r-spatial.github.io/sf/reference/geos_measures.html)
according to the R-spatial evolution, see more at
<https://r-spatial.org/r/2022/04/12/evolution.html>.

### Notes on arguments

The `DT` must be a `data.table`. If your data is a `data.frame`, you can
convert it by reference using
[`data.table::setDT()`](https://rdrr.io/pkg/data.table/man/setDT.html).

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
[`adehabitatHR::kernelUD()`](https://rdrr.io/pkg/adehabitatHR/man/kernelUD.html)
or
[`adehabitatHR::mcp()`](https://rdrr.io/pkg/adehabitatHR/man/mcp.html).

The `splitBy` argument offers further control over grouping. If within
your `DT`, you have multiple populations, subgroups or other distinct
parts, you can provide the name of the column which identifies them to
`splitBy`. The grouping performed by `group_polys` will only consider
rows within each `splitBy` subgroup.

## See also

[`build_polys()`](https://docs.ropensci.org/spatsoc/reference/build_polys.md)
[`group_times()`](https://docs.ropensci.org/spatsoc/reference/group_times.md)

Other Spatial grouping:
[`group_lines()`](https://docs.ropensci.org/spatsoc/reference/group_lines.md),
[`group_pts()`](https://docs.ropensci.org/spatsoc/reference/group_pts.md)

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

group_polys(DT, area = FALSE, hrType = 'mcp',
            hrParams = list(percent = 95), crs = utm,
            id = 'ID', coords = c('X', 'Y'))
#>            ID        X       Y            datetime population group
#>        <char>    <num>   <num>              <POSc>      <int> <int>
#>     1:      A 715851.4 5505340 2016-11-01 00:00:54          1     1
#>     2:      A 715822.8 5505289 2016-11-01 02:01:22          1     1
#>     3:      A 715872.9 5505252 2016-11-01 04:01:24          1     1
#>     4:      A 715820.5 5505231 2016-11-01 06:01:05          1     1
#>     5:      A 715830.6 5505227 2016-11-01 08:01:11          1     1
#>    ---                                                             
#> 14293:      J 700616.5 5509069 2017-02-28 14:00:54          1     1
#> 14294:      J 700622.6 5509065 2017-02-28 16:00:11          1     1
#> 14295:      J 700657.5 5509277 2017-02-28 18:00:55          1     1
#> 14296:      J 700610.3 5509269 2017-02-28 20:00:48          1     1
#> 14297:      J 700744.0 5508782 2017-02-28 22:00:39          1     1

areaDT <- group_polys(DT, area = TRUE, hrType = 'mcp',
                      hrParams = list(percent = 95), crs = utm,
                      id = 'ID', coords = c('X', 'Y'))
print(areaDT)
#>         ID1    ID2            area           proportion
#>      <char> <char>         <units>              <units>
#>   1:      A      A 145142204 [m^2] 100.000000 [percent]
#>   2:      B      A   9731424 [m^2]  91.676823 [percent]
#>   3:      C      A 110069503 [m^2]  99.961124 [percent]
#>   4:      E      A  60057115 [m^2]  96.286738 [percent]
#>   5:      F      A  87608610 [m^2]  93.849807 [percent]
#>   6:      G      A  12182087 [m^2]  93.204824 [percent]
#>   7:      H      A  61089896 [m^2]  75.352709 [percent]
#>   8:      I      A 125675100 [m^2]  99.939817 [percent]
#>   9:      J      A 101995862 [m^2]  76.979490 [percent]
#>  10:      A      B   9731424 [m^2]   6.704751 [percent]
#>  11:      B      B  10614922 [m^2] 100.000000 [percent]
#>  12:      C      B   9554942 [m^2]   8.677452 [percent]
#>  13:      E      B   9906676 [m^2]  15.882906 [percent]
#>  14:      F      B   9509693 [m^2]  10.187159 [percent]
#>  15:      G      B  10573553 [m^2]  80.897972 [percent]
#>  16:      H      B  10145705 [m^2]  12.514448 [percent]
#>  17:      I      B   9711953 [m^2]   7.723175 [percent]
#>  18:      J      B  10384186 [m^2]   7.837272 [percent]
#>  19:      A      C 110069503 [m^2]  75.835629 [percent]
#>  20:      B      C   9554942 [m^2]  90.014246 [percent]
#>  21:      C      C 110112310 [m^2] 100.000000 [percent]
#>  22:      E      C  54497018 [m^2]  87.372497 [percent]
#>  23:      F      C  82198072 [m^2]  88.053825 [percent]
#>  24:      G      C  12003003 [m^2]  91.834655 [percent]
#>  25:      H      C  55343937 [m^2]  68.265227 [percent]
#>  26:      I      C 109925578 [m^2]  87.415423 [percent]
#>  27:      J      C  88882259 [m^2]  67.082241 [percent]
#>  28:      D      D  11667275 [m^2] 100.000000 [percent]
#>  29:      A      E  60057115 [m^2]  41.378120 [percent]
#>  30:      B      E   9906676 [m^2]  93.327825 [percent]
#>  31:      C      E  54497018 [m^2]  49.492212 [percent]
#>  32:      E      E  62373195 [m^2] 100.000000 [percent]
#>  33:      F      E  45418786 [m^2]  48.654399 [percent]
#>  34:      G      E  12323593 [m^2]  94.287482 [percent]
#>  35:      H      E  61624267 [m^2]  76.011842 [percent]
#>  36:      I      E  56208280 [m^2]  44.698155 [percent]
#>  37:      J      E  61868357 [m^2]  46.693998 [percent]
#>  38:      A      F  87608610 [m^2]  60.360535 [percent]
#>  39:      B      F   9509693 [m^2]  89.587964 [percent]
#>  40:      C      F  82198072 [m^2]  74.649303 [percent]
#>  41:      E      F  45418786 [m^2]  72.817795 [percent]
#>  42:      F      F  93349803 [m^2] 100.000000 [percent]
#>  43:      G      F  11958427 [m^2]  91.493607 [percent]
#>  44:      H      F  46287984 [m^2]  57.094957 [percent]
#>  45:      I      F  83430540 [m^2]  66.345942 [percent]
#>  46:      J      F  61213465 [m^2]  46.199730 [percent]
#>  47:      A      G  12182087 [m^2]   8.393208 [percent]
#>  48:      B      G  10573553 [m^2]  99.610273 [percent]
#>  49:      C      G  12003003 [m^2]  10.900691 [percent]
#>  50:      E      G  12323593 [m^2]  19.757836 [percent]
#>  51:      F      G  11958427 [m^2]  12.810340 [percent]
#>  52:      G      G  13070233 [m^2] 100.000000 [percent]
#>  53:      H      G  12563547 [m^2]  15.496790 [percent]
#>  54:      I      G  12162428 [m^2]   9.671850 [percent]
#>  55:      J      G  12803712 [m^2]   9.663365 [percent]
#>  56:      A      H  61089896 [m^2]  42.089685 [percent]
#>  57:      B      H  10145705 [m^2]  95.579643 [percent]
#>  58:      C      H  55343937 [m^2]  50.261353 [percent]
#>  59:      E      H  61624267 [m^2]  98.799279 [percent]
#>  60:      F      H  46287984 [m^2]  49.585518 [percent]
#>  61:      G      H  12563547 [m^2]  96.123361 [percent]
#>  62:      H      H  81071930 [m^2] 100.000000 [percent]
#>  63:      I      H  57514743 [m^2]  45.737086 [percent]
#>  64:      J      H  66161291 [m^2]  49.934010 [percent]
#>  65:      A      I 125675100 [m^2]  86.587565 [percent]
#>  66:      B      I   9711953 [m^2]  91.493392 [percent]
#>  67:      C      I 109925578 [m^2]  99.830416 [percent]
#>  68:      E      I  56208280 [m^2]  90.116082 [percent]
#>  69:      F      I  83430540 [m^2]  89.374094 [percent]
#>  70:      G      I  12162428 [m^2]  93.054406 [percent]
#>  71:      H      I  57514743 [m^2]  70.942856 [percent]
#>  72:      I      I 125750781 [m^2] 100.000000 [percent]
#>  73:      J      I  93471355 [m^2]  70.545776 [percent]
#>  74:      A      J 101995862 [m^2]  70.273056 [percent]
#>  75:      B      J  10384186 [m^2]  97.826298 [percent]
#>  76:      C      J  88882259 [m^2]  80.719639 [percent]
#>  77:      E      J  61868357 [m^2]  99.190617 [percent]
#>  78:      F      J  61213465 [m^2]  65.574284 [percent]
#>  79:      G      J  12803712 [m^2]  97.960856 [percent]
#>  80:      H      J  66161291 [m^2]  81.608136 [percent]
#>  81:      I      J  93471355 [m^2]  74.330636 [percent]
#>  82:      J      J 132497451 [m^2] 100.000000 [percent]
#>  83:      A      D         0 [m^2]   0.000000 [percent]
#>  84:      B      D         0 [m^2]   0.000000 [percent]
#>  85:      C      D         0 [m^2]   0.000000 [percent]
#>  86:      D      A         0 [m^2]   0.000000 [percent]
#>  87:      D      B         0 [m^2]   0.000000 [percent]
#>  88:      D      C         0 [m^2]   0.000000 [percent]
#>  89:      D      E         0 [m^2]   0.000000 [percent]
#>  90:      D      F         0 [m^2]   0.000000 [percent]
#>  91:      D      G         0 [m^2]   0.000000 [percent]
#>  92:      D      H         0 [m^2]   0.000000 [percent]
#>  93:      D      I         0 [m^2]   0.000000 [percent]
#>  94:      D      J         0 [m^2]   0.000000 [percent]
#>  95:      E      D         0 [m^2]   0.000000 [percent]
#>  96:      F      D         0 [m^2]   0.000000 [percent]
#>  97:      G      D         0 [m^2]   0.000000 [percent]
#>  98:      H      D         0 [m^2]   0.000000 [percent]
#>  99:      I      D         0 [m^2]   0.000000 [percent]
#> 100:      J      D         0 [m^2]   0.000000 [percent]
#>         ID1    ID2            area           proportion
#>      <char> <char>         <units>              <units>
```
