# Calculate distance

**Internal function** - not developed to be used outside of spatsoc
functions

## Usage

``` r
calc_distance(geometry_a, geometry_b, x_a, y_a, x_b, y_b, crs, use_dist)
```

## Arguments

- geometry_a, geometry_b:

  sfc (simple feature geometry list column) from
  [`get_geometry()`](https://docs.ropensci.org/spatsoc/reference/get_geometry.md)

- x_a, x_b:

  X coordinate column, numeric

- y_a, y_b:

  Y coordinate column, numeric

- crs:

  crs for x_a, y_a (and if provided, x_b, y_b) coordinates, ignored for
  geometry_a and geometry_b arguments

- use_dist:

  boolean predetermine if distance calculated via dist

## Value

The underlying distance function used depends on the crs of the
coordinates or geometry provided.

- If the crs is longlat degrees (as determined by
  [`sf::st_is_longlat()`](https://r-spatial.github.io/sf/reference/st_is_longlat.html)),
  the distance function is
  [`sf::st_distance()`](https://r-spatial.github.io/sf/reference/geos_measures.html)
  which passes to
  [`s2::s2_distance()`](https://r-spatial.github.io/s2/reference/s2_is_collection.html)
  if
  [`sf::sf_use_s2()`](https://r-spatial.github.io/sf/reference/s2.html)
  is TRUE and
  [`lwgeom::st_geod_distance()`](https://r-spatial.github.io/lwgeom/reference/geod.html)
  if
  [`sf::sf_use_s2()`](https://r-spatial.github.io/sf/reference/s2.html)
  is FALSE. The distance returned has units set according to the crs.

- If the crs is not longlat degrees (eg. NULL, NA_crs\_, or projected),
  the distance function used is
  [`stats::dist()`](https://rdrr.io/r/stats/dist.html) (or Euclidean
  distance for pairwise distances), maintaining expected behaviour from
  previous versions. The distance returned does not have units set.

Note: in both cases, if the coordinates are NA then the result will be
NA.

## Details

Calculate distance for one of the following combinations:

- the distance matrix of points in geometry_a

- the distance matrix of points in x_a, y_a

- the pairwise distance between points in geometry_a and geometry_b

- the pairwise distance between points in x_a, y_a and x_b, y_b

Requirements:

- matching length between a and b objects if b provided

## Examples

``` r
# Load data.table
library(data.table)

# Example points
example <- data.table(
  X = c(0, 5, 5, 0, 0, NA_real_, 0,        NA_real_),
  Y = c(0, 0, 5, 5, 0, 0,        NA_real_, NA_real_)
)
# E, N, W, S
example[, spatsoc:::calc_distance(x_a = X, y_a = Y, crs = 4326, use_dist = FALSE)]
#> Units: [m]
#>          [,1]     [,2]     [,3]     [,4]     [,5] [,6] [,7] [,8]
#> [1,]      0.0 555975.5 785768.5 555975.5      0.0   NA   NA   NA
#> [2,] 555975.5      0.0 555975.5 785768.5 555975.5   NA   NA   NA
#> [3,] 785768.5 555975.5      0.0 553858.5 785768.5   NA   NA   NA
#> [4,] 555975.5 785768.5 553858.5      0.0 555975.5   NA   NA   NA
#> [5,]      0.0 555975.5 785768.5 555975.5      0.0   NA   NA   NA
#> [6,]       NA       NA       NA       NA       NA   NA   NA   NA
#> [7,]       NA       NA       NA       NA       NA   NA   NA   NA
#> [8,]       NA       NA       NA       NA       NA   NA   NA   NA
```
