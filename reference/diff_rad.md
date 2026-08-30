# Difference of two angles measured in radians

**Internal function** - not developed to be used outside of spatsoc
functions

## Usage

``` r
diff_rad(x, y, signed = FALSE, return_units = FALSE)
```

## Arguments

- x:

  angle in radians

- y:

  angle in radians

- signed:

  logical if signed difference should be returned, default FALSE

- return_units:

  return difference with units = 'rad'

## Value

Difference between x and y in radians. If signed is TRUE, the signed
difference is returned. If signed is FALSE, the absolute difference is
returned. Note: The difference is the smallest difference, eg. the
difference between 2 rad and -2.5 rad is 1.78.

## References

adapted from https://stackoverflow.com/a/7869457

## Examples

``` r
# Load data.table, units
library(data.table)
library(units)
#> udunits database from /usr/share/xml/udunits/udunits2.xml

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

# Set order using data.table::setorder
setorder(DT, datetime)

# Calculate direction
direction_step(
  DT = DT,
  id = 'ID',
  coords = c('X', 'Y'),
  crs = 32736
)
#>            ID        X       Y            datetime population        direction
#>        <char>    <num>   <num>              <POSc>      <int>          <units>
#>     1:      I 711042.0 5506384 2016-11-01 00:00:24          1  1.2185092 [rad]
#>     2:      C 710205.4 5505888 2016-11-01 00:00:44          1  0.1384333 [rad]
#>     3:      D 700875.0 5490954 2016-11-01 00:00:47          1  3.0618533 [rad]
#>     4:      E 701671.9 5504286 2016-11-01 00:00:48          1 -2.9977039 [rad]
#>     5:      F 705583.0 5513813 2016-11-01 00:00:48          1  1.3261714 [rad]
#>    ---                                                                        
#> 14293:      E 698956.7 5508224 2017-02-28 22:00:44          1         NA [rad]
#> 14294:      G 698307.6 5509182 2017-02-28 22:00:46          1         NA [rad]
#> 14295:      B 699759.4 5507878 2017-02-28 22:00:48          1         NA [rad]
#> 14296:      F 702841.7 5508583 2017-02-28 22:00:53          1         NA [rad]
#> 14297:      A 702780.3 5508592 2017-02-28 22:02:18          1         NA [rad]

# Differences
spatsoc:::diff_rad(DT[1, direction], DT[2, direction])
#> [1] 1.080076

# Note smallest difference returned
spatsoc:::diff_rad(as_units(2, 'rad'), as_units(-2.5, 'rad'))
#> [1] 1.783185
```
