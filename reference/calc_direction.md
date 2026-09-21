# Calculate direction

**Internal function** - not developed to be used outside of spatsoc
functions

## Usage

``` r
calc_direction(geometry_a, geometry_b, x_a, y_a, x_b, y_b, crs, use_transform)
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

- use_transform:

  boolean predetermine if coordinates/geometry need transform

## Value

Direction in units of radians, in range of pi, -pi where North = 0.

## Details

Calculate direction using
[`lwgeom::st_geod_azimuth()`](https://r-spatial.github.io/lwgeom/reference/st_geod_azimuth.html)
between one of:

- the sequence of points in geometry_a

- the sequence of points in x_a, y_a

- the pairwise points in geometry_a and geometry_b

- the pairwise points in x_a, y_a and x_b, y_b

Requirements:

- matching length between a and b objects if b provided

- crs is provided. If crs is not longlat (as determined by
  [`sf::st_is_longlat()`](https://r-spatial.github.io/sf/reference/st_is_longlat.html)),
  the coordinates or geometry will be transformed to `sf::st_crs(4326)`

- no missing values in coordinates

## Examples

``` r
# Load data.table
library(data.table)

# Example result for East, North, West, South directions
example <- data.table(
  X = c(0, 5, 5, 0, 0),
  Y = c(0, 0, 5, 5, 0)
)
# E, N, W, S
example[, spatsoc:::calc_direction(x_a = X, y_a = Y, crs = 4326, use_transform = FALSE)]
#> Linking to GEOS 3.12.1, GDAL 3.8.4, PROJ 9.4.0; sf_use_s2() is TRUE
#> Units: [rad]
#> [1]  1.570796  0.000000 -1.566991  3.141593
```
