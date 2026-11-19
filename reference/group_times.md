# Group times

`group_times` groups rows into time groups. The function expects date
time formatted data and a threshold argument. The threshold argument is
used to specify a time window within which rows are grouped.

## Usage

``` r
group_times(DT = NULL, datetime = NULL, threshold = NULL)
```

## Arguments

- DT:

  input data.table

- datetime:

  name of date time column(s). either 1 POSIXct or 2 IDate and ITime.
  e.g.: 'datetime' or c('idate', 'itime')

- threshold:

  threshold for grouping times. e.g.: '2 hours', '10 minutes', etc. if
  not provided, times will be matched exactly. Note that provided
  threshold must be in the expected format: '## unit'

## Value

`group_times` returns the input `DT` appended with a `timegroup` column
and additional temporal grouping columns to help investigate,
troubleshoot and interpret the timegroup.

The actual value of `timegroup` is arbitrary and represents the identity
of a given `timegroup` which 1 or more individuals are assigned to. If
the data was reordered, the group may change, but the contents of each
group would not.

The temporal grouping columns added depend on the `threshold` provided:

- `threshold` with unit minutes: "minutes" column added identifying the
  nearest minute group for each row.

- `threshold` with unit hours: "hours" column added identifying the
  nearest hour group for each row.

- `threshold` with unit days: "block" columns added identifying the
  multiday block for each row.

A message is returned when any of these columns already exist in the
input `DT`, because they will be overwritten.

See details for appending outputs using modify-by-reference in the
[FAQ](https://docs.ropensci.org/spatsoc/articles/faq.html).

## Details

The `DT` must be a `data.table`. If your data is a `data.frame`, you can
convert it by reference using
[`data.table::setDT()`](https://rdrr.io/pkg/data.table/man/setDT.html).

The `datetime` argument expects the name of a column in `DT` which is of
type `POSIXct` or the name of two columns in `DT` which are of type
`IDate` and `ITime`.

`threshold` must be provided in units of minutes, hours or days. The
character string should start with an integer followed by a unit,
separated by a space. It is interpreted in terms of 24 hours which poses
the following limitations:

- minutes, hours and days cannot be fractional

- minutes must divide evenly into 60

- minutes must not exceed 60

- minutes, hours which are nearer to the next day, are grouped as such

- hours must divide evenly into 24

- multi-day blocks should divide into the range of days, else the blocks
  may not be the same length

In addition, the `threshold` is considered a fixed window throughout the
time series and the rows are grouped to the nearest interval.

If `threshold` is NULL, rows are grouped using the `datetime` column
directly.

## See also

`group_pts` `group_lines` `group_polys`

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

group_times(DT, datetime = 'datetime', threshold = '5 minutes')
#>            ID        X       Y            datetime population minutes timegroup
#>        <char>    <num>   <num>              <POSc>      <int>   <int>     <int>
#>     1:      A 715851.4 5505340 2016-11-01 00:00:54          1       0         1
#>     2:      A 715822.8 5505289 2016-11-01 02:01:22          1       0         2
#>     3:      A 715872.9 5505252 2016-11-01 04:01:24          1       0         3
#>     4:      A 715820.5 5505231 2016-11-01 06:01:05          1       0         4
#>     5:      A 715830.6 5505227 2016-11-01 08:01:11          1       0         5
#>    ---                                                                         
#> 14293:      J 700616.5 5509069 2017-02-28 14:00:54          1       0      1393
#> 14294:      J 700622.6 5509065 2017-02-28 16:00:11          1       0      1394
#> 14295:      J 700657.5 5509277 2017-02-28 18:00:55          1       0      1449
#> 14296:      J 700610.3 5509269 2017-02-28 20:00:48          1       0      1395
#> 14297:      J 700744.0 5508782 2017-02-28 22:00:39          1       0      1396

group_times(DT, datetime = 'datetime', threshold = '2 hours')
#> minutes, timegroup columns found in input DT and will be overwritten by this function
#>            ID        X       Y            datetime population hours timegroup
#>        <char>    <num>   <num>              <POSc>      <int> <int>     <int>
#>     1:      A 715851.4 5505340 2016-11-01 00:00:54          1     0         1
#>     2:      A 715822.8 5505289 2016-11-01 02:01:22          1     2         2
#>     3:      A 715872.9 5505252 2016-11-01 04:01:24          1     4         3
#>     4:      A 715820.5 5505231 2016-11-01 06:01:05          1     6         4
#>     5:      A 715830.6 5505227 2016-11-01 08:01:11          1     8         5
#>    ---                                                                       
#> 14293:      J 700616.5 5509069 2017-02-28 14:00:54          1    14      1393
#> 14294:      J 700622.6 5509065 2017-02-28 16:00:11          1    16      1394
#> 14295:      J 700657.5 5509277 2017-02-28 18:00:55          1    18      1440
#> 14296:      J 700610.3 5509269 2017-02-28 20:00:48          1    20      1395
#> 14297:      J 700744.0 5508782 2017-02-28 22:00:39          1    22      1396

group_times(DT, datetime = 'datetime', threshold = '10 days')
#> hours, timegroup columns found in input DT and will be overwritten by this function
#> Warning: the minimum and maximum days in DT are not evenly divisible by the provided block length
#>            ID        X       Y            datetime population block timegroup
#>        <char>    <num>   <num>              <POSc>      <int> <int>     <int>
#>     1:      A 715851.4 5505340 2016-11-01 00:00:54          1    31         1
#>     2:      A 715822.8 5505289 2016-11-01 02:01:22          1    31         1
#>     3:      A 715872.9 5505252 2016-11-01 04:01:24          1    31         1
#>     4:      A 715820.5 5505231 2016-11-01 06:01:05          1    31         1
#>     5:      A 715830.6 5505227 2016-11-01 08:01:11          1    31         1
#>    ---                                                                       
#> 14293:      J 700616.5 5509069 2017-02-28 14:00:54          1     6        13
#> 14294:      J 700622.6 5509065 2017-02-28 16:00:11          1     6        13
#> 14295:      J 700657.5 5509277 2017-02-28 18:00:55          1     6        13
#> 14296:      J 700610.3 5509269 2017-02-28 20:00:48          1     6        13
#> 14297:      J 700744.0 5508782 2017-02-28 22:00:39          1     6        13
```
