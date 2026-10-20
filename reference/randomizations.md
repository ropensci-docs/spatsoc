# Data-stream randomizations

`randomizations` performs data-stream social network randomization. The
function expects a `data.table` with relocation data, individual
identifiers and a randomization `type`. The `data.table` is randomized
either using `step` or `daily` between-individual methods, or
within-individual daily `trajectory` method described by Spiegel et al.
(2016).

## Usage

``` r
randomizations(
  DT = NULL,
  type = NULL,
  id = NULL,
  group = NULL,
  coords = NULL,
  datetime = NULL,
  splitBy = NULL,
  iterations = NULL
)
```

## Arguments

- DT:

  input data.table

- type:

  one of 'daily', 'step' or 'trajectory' - see details

- id:

  character string of ID column name

- group:

  generated from spatial grouping functions - see details

- coords:

  character vector of X coordinate and Y coordinate column names. Note:
  the order is assumed X followed by Y column names

- datetime:

  field used for providing date time or time group - see details

- splitBy:

  List of fields in DT to split the randomization process by

- iterations:

  The number of iterations to randomize

## Value

`randomizations` returns the random date time or random id along with
the original `DT`, depending on the randomization `type`. The length of
the returned `data.table` is the original number of rows multiplied by
the number of iterations + 1. For example, 3 iterations will return 4x -
one observed and three randomized.

Two columns are always returned:

- observed - if the rows represent the observed (TRUE/FALSE)

- iteration - iteration of rows (where 0 is the observed)

In addition, depending on the randomization type, random ID or random
date time columns are returned:

- step - `randomID` each time step

- daily - `randomID` for each day and `jul` indicating julian day

- trajectory - a random date time ("random" prefixed to `datetime`
  argument), observed `jul` and `randomJul` indicating the random day
  relocations are swapped to.

## Details

The `DT` must be a `data.table`. If your data is a `data.frame`, you can
convert it by reference using
[`data.table::setDT()`](https://rdrr.io/pkg/data.table/man/setDT.html).

Three randomization `type`s are provided:

1.  step - randomizes identities of relocations between individuals
    within each time step.

2.  daily - randomizes identities of relocations between individuals
    within each day.

3.  trajectory - randomizes daily trajectories within individuals
    (Spiegel et al. 2016).

Depending on the `type`, the `datetime` must be a certain format:

- step - datetime is integer group created by `group_times`

- daily - datetime is `POSIXct` format

- trajectory - datetime is `POSIXct` format

The `id`, `datetime`, (and optional `splitBy`) arguments expect the
names of respective columns in `DT` which correspond to the individual
identifier, date time, and additional grouping columns. The `coords`
argument is only required when the `type` is "trajectory", since the
coordinates are required for recalculating spatial groups with
`group_pts`, `group_lines` or `group_polys`.

Please note that if the data extends over multiple years, a column
indicating the year should be provided to the `splitBy` argument. This
will ensure randomizations only occur within each year.

The `group` argument is expected only when `type` is 'step' or 'daily'.

For example, using
[`data.table::year()`](https://rdrr.io/pkg/data.table/man/IDateTime.html):

     DT[, yr := year(datetime)] randomizations(DT, type = 'step',
    id = 'ID', datetime = 'timegroup', splitBy = 'yr') 

`iterations` is set to 1 if not provided. Take caution with a large
value for `iterations` with large input `DT`.

## References

<doi:10.1111/2041-210X.12553>

## See also

Other Social network tools:
[`get_gbi()`](https://docs.ropensci.org/spatsoc/reference/get_gbi.md)

## Examples

``` r
# Load data.table
library(data.table)

# Read example data
DT <- fread(system.file("extdata", "DT.csv", package = "spatsoc"))

# Select only individuals A, B, C for this example
DT <- DT[ID %in% c('A', 'B', 'C')]

# Date time columns
DT[, datetime := as.POSIXct(datetime)]
#>           ID        X       Y            datetime population
#>       <char>    <num>   <num>              <POSc>      <int>
#>    1:      A 715851.4 5505340 2016-11-01 00:00:54          1
#>    2:      A 715822.8 5505289 2016-11-01 02:01:22          1
#>    3:      A 715872.9 5505252 2016-11-01 04:01:24          1
#>    4:      A 715820.5 5505231 2016-11-01 06:01:05          1
#>    5:      A 715830.6 5505227 2016-11-01 08:01:11          1
#>   ---                                                       
#> 4265:      C 702093.6 5510180 2017-02-28 14:00:44          1
#> 4266:      C 702086.0 5510183 2017-02-28 16:00:42          1
#> 4267:      C 702961.8 5509447 2017-02-28 18:00:53          1
#> 4268:      C 703130.4 5509528 2017-02-28 20:00:54          1
#> 4269:      C 702872.3 5508531 2017-02-28 22:00:18          1
DT[, yr := year(datetime)]
#>           ID        X       Y            datetime population    yr
#>       <char>    <num>   <num>              <POSc>      <int> <int>
#>    1:      A 715851.4 5505340 2016-11-01 00:00:54          1  2016
#>    2:      A 715822.8 5505289 2016-11-01 02:01:22          1  2016
#>    3:      A 715872.9 5505252 2016-11-01 04:01:24          1  2016
#>    4:      A 715820.5 5505231 2016-11-01 06:01:05          1  2016
#>    5:      A 715830.6 5505227 2016-11-01 08:01:11          1  2016
#>   ---                                                             
#> 4265:      C 702093.6 5510180 2017-02-28 14:00:44          1  2017
#> 4266:      C 702086.0 5510183 2017-02-28 16:00:42          1  2017
#> 4267:      C 702961.8 5509447 2017-02-28 18:00:53          1  2017
#> 4268:      C 703130.4 5509528 2017-02-28 20:00:54          1  2017
#> 4269:      C 702872.3 5508531 2017-02-28 22:00:18          1  2017

# Temporal grouping
group_times(DT, datetime = 'datetime', threshold = '5 minutes')
#>           ID        X       Y            datetime population    yr minutes
#>       <char>    <num>   <num>              <POSc>      <int> <int>   <int>
#>    1:      A 715851.4 5505340 2016-11-01 00:00:54          1  2016       0
#>    2:      A 715822.8 5505289 2016-11-01 02:01:22          1  2016       0
#>    3:      A 715872.9 5505252 2016-11-01 04:01:24          1  2016       0
#>    4:      A 715820.5 5505231 2016-11-01 06:01:05          1  2016       0
#>    5:      A 715830.6 5505227 2016-11-01 08:01:11          1  2016       0
#>   ---                                                                     
#> 4265:      C 702093.6 5510180 2017-02-28 14:00:44          1  2017       0
#> 4266:      C 702086.0 5510183 2017-02-28 16:00:42          1  2017       0
#> 4267:      C 702961.8 5509447 2017-02-28 18:00:53          1  2017       0
#> 4268:      C 703130.4 5509528 2017-02-28 20:00:54          1  2017       0
#> 4269:      C 702872.3 5508531 2017-02-28 22:00:18          1  2017       0
#>       timegroup
#>           <int>
#>    1:         1
#>    2:         2
#>    3:         3
#>    4:         4
#>    5:         5
#>   ---          
#> 4265:      1393
#> 4266:      1394
#> 4267:      1449
#> 4268:      1395
#> 4269:      1396

# Spatial grouping with timegroup
group_pts(DT, threshold = 5, id = 'ID',
          coords = c('X', 'Y'), timegroup = 'timegroup')
#>           ID        X       Y            datetime population    yr minutes
#>       <char>    <num>   <num>              <POSc>      <int> <int>   <int>
#>    1:      A 715851.4 5505340 2016-11-01 00:00:54          1  2016       0
#>    2:      A 715822.8 5505289 2016-11-01 02:01:22          1  2016       0
#>    3:      A 715872.9 5505252 2016-11-01 04:01:24          1  2016       0
#>    4:      A 715820.5 5505231 2016-11-01 06:01:05          1  2016       0
#>    5:      A 715830.6 5505227 2016-11-01 08:01:11          1  2016       0
#>   ---                                                                     
#> 4265:      C 702093.6 5510180 2017-02-28 14:00:44          1  2017       0
#> 4266:      C 702086.0 5510183 2017-02-28 16:00:42          1  2017       0
#> 4267:      C 702961.8 5509447 2017-02-28 18:00:53          1  2017       0
#> 4268:      C 703130.4 5509528 2017-02-28 20:00:54          1  2017       0
#> 4269:      C 702872.3 5508531 2017-02-28 22:00:18          1  2017       0
#>       timegroup group
#>           <int> <int>
#>    1:         1     1
#>    2:         2     2
#>    3:         3     3
#>    4:         4     4
#>    5:         5     5
#>   ---                
#> 4265:      1393  4228
#> 4266:      1394  4229
#> 4267:      1449  4230
#> 4268:      1395  4231
#> 4269:      1396  4232

# Randomization: step
randStep <- randomizations(
    DT,
    type = 'step',
    id = 'ID',
    group = 'group',
    datetime = 'timegroup',
    splitBy = 'yr',
    iterations = 2
)

# Randomization: daily
randDaily <- randomizations(
    DT,
    type = 'daily',
    id = 'ID',
    group = 'group',
    datetime = 'datetime',
    splitBy = 'yr',
    iterations = 2
)

# Randomization: trajectory
randTraj <- randomizations(
    DT,
    type = 'trajectory',
    id = 'ID',
    group = NULL,
    coords = c('X', 'Y'),
    datetime = 'datetime',
    splitBy = 'yr',
    iterations = 2
)
```
