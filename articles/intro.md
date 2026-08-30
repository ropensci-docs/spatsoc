# Introduction to spatsoc

The `spatsoc` package provides functionality for analyzing animal
relocation data in time and space to identify potential interactions
among individuals and build gambit-of-the-group data for constructing
social networks.

The package contains grouping and edge-list generating functions that
are used for identifying spatially and temporally explicit groups from
input data. In addition, we provide social network analysis functions
for randomizing individual identifiers within groups, designed to test
whether social networks generated from animal relocation data were based
on non-random social proximity among individuals and for generating
group by individual matrices.

The functions were developed for application across animal relocation
data, for example, proximity based social network analyses and spatial
and temporal clustering of points.

------------------------------------------------------------------------

See the other vignettes for further information:

- [Introduction to
  spatsoc](https://docs.ropensci.org/spatsoc/articles/intro.html)
  - temporal grouping
  - spatiotemporal grouping with `group_pts`, `group_lines`,
    `group_polys`
  - distance based edge-list generation with `edge_dist`
- [Frequently asked questions about
  spatsoc](https://docs.ropensci.org/spatsoc/articles/faq.html)
  - install
  - function details for `group_times`, `group_pts`, `group_lines`,
    `group_polys`, `edge_dist`, `edge_nn`, and `randomizations`
  - package design including modify-by-reference, data.table column
    allocation
  - calculating summary information
- [Using spatsoc in social network
  analysis](https://docs.ropensci.org/spatsoc/articles/using-in-sna.html)
  - generating gambit-of-the-group data
  - generating observed networks
  - data stream randomization, randomized networks
  - network metrics
- [Using distance based edge-lists generating functions, dyad_id, and
  fusion_id](https://docs.ropensci.org/spatsoc/articles/using-edge-and-dyad.html)
  - generate distance based edge-lists with `edge_dist` and `edge_nn`
  - generate dyad identifiers for edge-lists with `dyad_id`
  - identify fusion events with `fusion_id`
- [Geometry
  interface](https://docs.ropensci.org/spatsoc/articles/geometry-interface-and-spatial-measures.html)
  - using `get_geometry` to setup a geometry column and use the geometry
    interface
  - details of underlying distance, direction and centroid spatial
    measures
  - converting to and from related packages
- [Interspecific
  interactions](https://docs.ropensci.org/spatsoc/articles/interspecific-interactions.html)
  - combine two movement datasets
  - identify interspecific interactions

## Data preparation

``` r

# Load packages
library(spatsoc)
library(data.table)
```

    ## 
    ## Attaching package: 'data.table'

    ## The following object is masked from 'package:base':
    ## 
    ##     %notin%

``` r

# Read in spatsoc's example data
DT <- fread(system.file("extdata", "DT.csv", package = "spatsoc"))

# Use subset of individuals
DT <- DT[ID %in% c('H', 'I', 'J')]

# Cast character column 'datetime' as POSIXct
DT[, datetime := as.POSIXct(datetime, tz = 'UTC')]
DT <- DT[ID %chin% c('H', 'I', 'J')]
```

| ID  |        X |       Y | datetime            | population |
|:----|---------:|--------:|:--------------------|-----------:|
| H   | 701724.1 | 5504325 | 2016-11-01 00:00:49 |          1 |
| H   | 701648.5 | 5504276 | 2016-11-01 02:00:33 |          1 |
| I   | 711042.0 | 5506384 | 2016-11-01 00:00:24 |          1 |
| I   | 711229.0 | 5506446 | 2016-11-01 02:00:33 |          1 |
| J   | 707568.6 | 5500406 | 2016-11-01 00:00:56 |          1 |
| J   | 707566.5 | 5500404 | 2016-11-01 02:00:21 |          1 |

`spatsoc` expects a `data.table` for all of its functions. If you have a
`data.frame`, you can use
[`data.table::setDT()`](https://rdrr.io/pkg/data.table/man/setDT.html)
to convert it by reference. If your data is a CSV, you can use
[`data.table::fread()`](https://rdrr.io/pkg/data.table/man/fread.html)
to import it as a `data.table`.

The data consist of relocations of 3 individuals over
`r DT[, max(yday(datetime)) - min(yday(datetime))]` days. Using these
data, we can compare the various grouping methods available in
`spatsoc`. Note: these examples will use a subset of the data, only
individuals H, I and J.

## Temporal grouping

The `group_times` function is used to group relocations temporally. It
is flexible to a threshold provided in units of minutes, hours or days.
Since GPS fixes taken at regular intervals have some level of
variability, we will provide a time threshold (`threshold`), to consider
all fixes within this threshold taken at the same time. Alternatively,
we may want to understand different scales of grouping, perhaps daily
movement trajectories or seasonal home range overlap.

``` r

group_times(DT, datetime = 'datetime', threshold = '5 minutes')
```

| ID  |        X |       Y | datetime            | minutes | timegroup |
|:----|---------:|--------:|:--------------------|--------:|----------:|
| I   | 711042.0 | 5506384 | 2016-11-01 00:00:24 |       0 |         1 |
| H   | 701724.1 | 5504325 | 2016-11-01 00:00:49 |       0 |         1 |
| J   | 707568.6 | 5500406 | 2016-11-01 00:00:56 |       0 |         1 |
| J   | 707566.5 | 5500404 | 2016-11-01 02:00:21 |       0 |         2 |
| H   | 701648.5 | 5504276 | 2016-11-01 02:00:33 |       0 |         2 |
| I   | 711229.0 | 5506446 | 2016-11-01 02:00:33 |       0 |         2 |
| J   | 707562.6 | 5500374 | 2016-11-01 04:00:41 |       0 |         3 |
| I   | 711124.0 | 5506407 | 2016-11-01 04:00:44 |       0 |         3 |
| H   | 701607.2 | 5504291 | 2016-11-01 04:00:54 |       0 |         3 |

A message is returned when `group_times` is run again on the same `DT`,
as the columns already exist in the input `DT` and will be overwritten.

``` r

group_times(DT, datetime = 'datetime', threshold = '2 hours')
```

    ## minutes, timegroup columns found in input DT and will be overwritten by this function

| ID  |        X |       Y | datetime            | hours | timegroup |
|:----|---------:|--------:|:--------------------|------:|----------:|
| I   | 711042.0 | 5506384 | 2016-11-01 00:00:24 |     0 |         1 |
| H   | 701724.1 | 5504325 | 2016-11-01 00:00:49 |     0 |         1 |
| J   | 707568.6 | 5500406 | 2016-11-01 00:00:56 |     0 |         1 |
| J   | 707566.5 | 5500404 | 2016-11-01 02:00:21 |     2 |         2 |
| H   | 701648.5 | 5504276 | 2016-11-01 02:00:33 |     2 |         2 |
| I   | 711229.0 | 5506446 | 2016-11-01 02:00:33 |     2 |         2 |
| J   | 707562.6 | 5500374 | 2016-11-01 04:00:41 |     4 |         3 |
| I   | 711124.0 | 5506407 | 2016-11-01 04:00:44 |     4 |         3 |
| H   | 701607.2 | 5504291 | 2016-11-01 04:00:54 |     4 |         3 |

``` r

group_times(DT, datetime = 'datetime', threshold = '5 days')
```

    ## hours, timegroup columns found in input DT and will be overwritten by this function

| ID  |        X |       Y | datetime            | block | timegroup |
|:----|---------:|--------:|:--------------------|------:|----------:|
| I   | 711064.1 | 5506363 | 2016-11-01 08:00:50 |    62 |         1 |
| J   | 707363.8 | 5500571 | 2016-11-03 16:00:41 |    62 |         1 |
| J   | 707053.2 | 5500681 | 2016-11-05 12:00:53 |    62 |         1 |
| I   | 705367.4 | 5505915 | 2016-11-09 10:00:41 |    63 |         2 |
| H   | 701378.0 | 5504512 | 2016-11-09 20:00:29 |    63 |         2 |
| I   | 701902.3 | 5505319 | 2016-11-10 04:00:24 |    63 |         2 |
| H   | 701672.1 | 5504792 | 2016-11-11 08:00:56 |    64 |         3 |
| H   | 701200.5 | 5504696 | 2016-11-12 22:00:54 |    64 |         3 |
| J   | 705791.6 | 5499354 | 2016-11-14 04:00:50 |    64 |         3 |

## Spatial grouping

The `group_pts` function compares the relocations of all individuals in
each timegroup and groups individuals based on a distance threshold
provided by the user. The `group_pts` function uses the “chain rule”
where three or more individuals that are all within the defined
threshold distance of at least one other individual are considered in
the same group. For point based spatial grouping with a distance
threshold that does not use the chain rule, see `edge_dist` below.

``` r

group_times(DT = DT, datetime = 'datetime', threshold = '15 minutes')
group_pts(DT, threshold = 50, id = 'ID', coords = c('X', 'Y'), timegroup = 'timegroup')
```

    ## block, timegroup columns found in input DT and will be overwritten by this function

| ID  |        X |       Y | timegroup | group |
|:----|---------:|--------:|----------:|------:|
| H   | 699126.1 | 5508836 |       771 |   771 |
| I   | 699130.0 | 5508761 |       771 |   771 |
| J   | 699138.0 | 5508797 |       771 |   771 |
| H   | 699930.5 | 5508032 |       772 |   772 |
| H   | 700139.2 | 5507325 |       773 |   773 |
| I   | 700131.7 | 5507321 |       773 |   773 |
| H   | 700012.2 | 5508010 |       774 |   774 |
| I   | 700015.0 | 5508001 |       774 |   774 |
| J   | 700002.3 | 5508005 |       774 |   774 |

The `group_lines` function groups individuals whose trajectories
intersect in a specified time interval. This represents a coarser
grouping method than `group_pts` which can help understand shared space
at daily, weekly or other temporal resolutions.

``` r

utm <- 32736

group_times(DT = DT, datetime = 'datetime', threshold = '1 day')
group_lines(DT, threshold = 50, crs = utm, 
            id = 'ID', coords = c('X', 'Y'),
            timegroup = 'timegroup', sortBy = 'datetime')
```

    ## minutes, timegroup columns found in input DT and will be overwritten by this function

    ## group column will be overwritten by this function

| ID  | timegroup | group |
|:----|----------:|------:|
| H   |         1 |     1 |
| I   |         1 |   121 |
| J   |         1 |   204 |
| H   |         2 |     2 |
| I   |         2 |   122 |
| J   |         2 |   205 |
| H   |         3 |     3 |
| I   |         3 |   123 |
| J   |         3 |   206 |

The `group_polys` function groups individuals whose home ranges
intersect. This represents the coarsest grouping method, to provide a
measure of overlap across seasons, years or all available relocations.
It can either return the proportion of home range area overlapping
between individuals or simple groups. Home ranges are calculated using
[`adehabitatHR::kernelUD`](https://rdrr.io/pkg/adehabitatHR/man/kernelUD.html)
or [`adehabitatHR::mcp`](https://rdrr.io/pkg/adehabitatHR/man/mcp.html).
Alternatively, a simple features POLYGON or MULTIPOLYGON object can be
provided to the `sfPolys` argument along with an id column.

``` r

utm <- 32736
group_times(DT = DT, datetime = 'datetime', threshold = '8 days')
group_polys(DT = DT, area = TRUE, hrType = 'mcp',
           hrParams = list('percent' = 95),
           crs = utm,
           coords = c('X', 'Y'), id = 'ID')
```

    ## timegroup columns found in input DT and will be overwritten by this function

    ## Registered S3 methods overwritten by 'adehabitatMA':
    ##   method                       from
    ##   print.SpatialPixelsDataFrame sp  
    ##   print.SpatialPixels          sp

| ID1 | ID2 |              area |            proportion |
|:----|:----|------------------:|----------------------:|
| H   | H   |  81071930 \[m^2\] | 100.00000 \[percent\] |
| I   | H   |  57514743 \[m^2\] |  45.73709 \[percent\] |
| J   | H   |  66161291 \[m^2\] |  49.93401 \[percent\] |
| H   | I   |  57514743 \[m^2\] |  70.94286 \[percent\] |
| I   | I   | 125750781 \[m^2\] | 100.00000 \[percent\] |
| J   | I   |  93471355 \[m^2\] |  70.54578 \[percent\] |
| H   | J   |  66161291 \[m^2\] |  81.60814 \[percent\] |
| I   | J   |  93471355 \[m^2\] |  74.33064 \[percent\] |
| J   | J   | 132497451 \[m^2\] | 100.00000 \[percent\] |

## Edge-list generation

The `edge_dist` function calculates the geographic distance between
between individuals within each timegroup and returns all paired
relocations within the spatial threshold. `edge_dist` uses a distance
matrix like group_pts, but, in contrast, does not use the chain rule to
group relocations.

``` r

group_times(DT = DT, datetime = 'datetime', threshold = '15 minutes')
edge_dist(DT, threshold = 50, id = 'ID', 
          coords = c('X', 'Y'), timegroup = 'timegroup', fillNA = TRUE)
```

    ## block, timegroup columns found in input DT and will be overwritten by this function

| timegroup | ID1 | ID2 |
|----------:|:----|:----|
|       158 | H   | NA  |
|       158 | I   | NA  |
|       158 | J   | NA  |
|       159 | H   | I   |
|       159 | I   | H   |
|       159 | J   | NA  |
|       160 | H   | I   |
|       160 | I   | H   |
|       160 | J   | NA  |

The `edge_nn` function calculates the nearest neighbour to each
individual within each time group. If the optional distance threshold is
provided, it is used to limit the maximum distance between neighbours.
`edge_nn` returns an edge list of each individual and their nearest
neighbour.

``` r

group_times(DT = DT, datetime = 'datetime', threshold = '15 minutes')
edge_nn(DT, id = 'ID', coords = c('X', 'Y'), timegroup = 'timegroup')
```

    ## minutes, timegroup columns found in input DT and will be overwritten by this function

| timegroup | ID  | NN  |
|----------:|:----|:----|
|         1 | H   | J   |
|         1 | I   | J   |
|         1 | J   | I   |
|         2 | H   | J   |
|         2 | I   | J   |
|         2 | J   | I   |
