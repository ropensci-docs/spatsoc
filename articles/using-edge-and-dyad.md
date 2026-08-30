# Using distance based edge-list generating functions, dyad_id and fusion_id

`spatsoc` can be used in social network analysis to generate distance
based edge-lists from GPS relocation data using either the `edge_dist`
or the `edge_nn` function.

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

## Generate edge-lists

spatsoc provides users with one temporal (`group_times`) and two
distance based edge-list generating functions (`edge_dist`, `edge_nn`)
to generate edge-lists from GPS relocations. Users can consider edges
defined by either the spatial proximity between individuals (with
`edge_dist`), by nearest neighbour (with `edge_nn`) or by nearest
neighbour with a maximum distance (with `edge_nn`). The edge-lists can
be used directly by the animal social network package `asnipe` to
generate networks.

### 1. Load packages and prepare data

`spatsoc` expects a `data.table` for all `DT` arguments and date time
columns to be formatted `POSIXct`.

``` r

## Load packages
library(spatsoc)
library(data.table)
#> 
#> Attaching package: 'data.table'
#> The following object is masked from 'package:base':
#> 
#>     %notin%
```

``` r

## Read data as a data.table
DT <- fread(system.file("extdata", "DT.csv", package = "spatsoc"))

## Cast datetime column to POSIXct
DT[, datetime := as.POSIXct(datetime)]
```

Next, we will group relocations temporally with `group_times` and
generate edges lists with one of `edge_dist`, `edge_dist`. Note: these
are mutually exclusive, only select one edge-list generating function at
a time.

### 2. a) `edge_dist`

Distance based edge-lists where relocations in each timegroup are
considered edges if they are within the spatial distance defined by the
user with the `threshold` argument. Depending on species and study
system, relevant temporal and spatial distance thresholds are used. In
this case, relocations within 5 minutes and 50 meters are considered
edges.

This is the non-chain rule implementation similar to `group_pts`. Edges
are defined by the distance threshold and NAs are returned for
individuals within each timegroup if they are not within the threshold
distance of any other individual (if `fillNA` is TRUE).

Optionally, `edge_dist` can return the distances between individuals
(less than the threshold) in a column named ‘distance’ with argument
`returnDist = TRUE`.

``` r

# Temporal groups
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

# Edge-list generation
edges <- edge_dist(
  DT,
  threshold = 100,
  id = 'ID',
  coords = c('X', 'Y'),
  timegroup = 'timegroup',
  returnDist = TRUE,
  fillNA = TRUE
)
```

### 2. b) `edge_nn`

Nearest neighbour based edge-lists where each individual is connected to
their nearest neighbour. `edge_nn` can be used to generate edge-lists
defined either by nearest neighbour or nearest neighbour with a maximum
distance. As with grouping functions and `edge_dist`, temporal and
spatial threshold depend on species and study system.

NAs are returned for nearest neighbour for an individual was alone in a
timegroup (and/or splitBy) or if the distance between an individual and
its nearest neighbour is greater than the threshold.

Optionally, `edge_nn` can return the distances between individuals (less
than the threshold) in a column named ‘distance’ with argument
`returnDist = TRUE`.

``` r

# Temporal groups
group_times(DT, datetime = 'datetime', threshold = '5 minutes')

# Edge-list generation
edges <- edge_nn(
  DT,
  id = 'ID',
  coords = c('X', 'Y'),
  timegroup = 'timegroup'
)

# Edge-list generation using maximum distance threshold
edges <- edge_nn(
  DT, 
  id = 'ID', 
  coords = c('X', 'Y'),
  timegroup = 'timegroup', 
  threshold = 100
)

# Edge-list generation using maximum distance threshold, returning distances
edges <- edge_nn(
  DT, 
  id = 'ID', 
  coords = c('X', 'Y'),
  timegroup = 'timegroup', 
  threshold = 100,
  returnDist = TRUE
)
```

## Dyads

### 3. `dyad_id`

The function `dyad_id` can be used to generate a unique, undirected dyad
identifier for edge-lists.

``` r

# In this case, using the edges generated in 2. a) edge_dist
dyad_id(edges, id1 = 'ID1', id2 = 'ID2')
#> Key: <timegroup, ID1>
#>        timegroup    ID1    ID2  distance dyadID
#>            <int> <char> <char>     <num> <char>
#>     1:         1      A   <NA>        NA   <NA>
#>     2:         1      B      G  5.782904    B-G
#>     3:         1      C   <NA>        NA   <NA>
#>     4:         1      D   <NA>        NA   <NA>
#>     5:         1      E      H 65.061671    E-H
#>    ---                                         
#> 22942:      1457      G   <NA>        NA   <NA>
#> 22943:      1458      H   <NA>        NA   <NA>
#> 22944:      1459      I   <NA>        NA   <NA>
#> 22945:      1460      J   <NA>        NA   <NA>
#> 22946:      1461      J   <NA>        NA   <NA>
```

Once we have generated dyad ids, we can measure consecutive relocations,
start and end relocation, etc. **Note:** since the edges are duplicated
A-B and B-A, you will need to use the unique timegroup\*dyadID or divide
counts by 2.

## Fusion events

### 4. `fusion_id`

The function `fusion_id` can be used to identify fusion events in
distance based edge-lists. The “n_min_length” argument defines the
minimum number of successive fixes that are required to establish a
fusion event. The “n_max_missing” argument defines the the maximum
number of allowable missing observations for the dyad within a fusion
event. The “allow_split” argument defines if a single observation can be
greater than the threshold distance without initiating fission event.

``` r

fusion_id(
  edges = edges,
  threshold = 100,
  n_min_length = 1,
  n_max_missing = 0,
  allow_split = FALSE
)

# Print first 10 fusion events
print(edges[fusionID <= 5])
#> Key: <timegroup, ID1>
#>     timegroup    ID1    ID2  distance dyadID fusionID
#>         <int> <char> <char>     <num> <char>    <int>
#>  1:         1      B      G  5.782904    B-G        1
#>  2:         1      E      H 65.061671    E-H        2
#>  3:         1      G      B  5.782904    B-G        1
#>  4:         1      H      E 65.061671    E-H        2
#>  5:         2      E      H 79.659918    E-H        2
#>  6:         2      H      E 79.659918    E-H        2
#>  7:         3      E      H 84.345303    E-H        2
#>  8:         3      H      E 84.345303    E-H        2
#>  9:         4      B      G 21.129001    B-G        3
#> 10:         4      E      H 11.961466    E-H        2
#> 11:         4      G      B 21.129001    B-G        3
#> 12:         4      H      E 11.961466    E-H        2
#> 13:         5      B      G  9.630521    B-G        3
#> 14:         5      E      H 31.498449    E-H        2
#> 15:         5      G      B  9.630521    B-G        3
#> 16:         5      H      E 31.498449    E-H        2
#> 17:         6      B      G 27.816927    B-G        3
#> 18:         6      G      B 27.816927    B-G        3
#> 19:         7      B      G 17.463267    B-G        3
#> 20:         7      G      B 17.463267    B-G        3
#> 21:         8      A      I 76.831235    A-I        4
#> 22:         8      B      G 51.547342    B-G        3
#> 23:         8      G      B 51.547342    B-G        3
#> 24:         8      I      A 76.831235    A-I        4
#> 25:         9      B      G 66.724550    B-G        3
#> 26:         9      G      B 66.724550    B-G        3
#> 27:        10      B      G 16.209695    B-G        3
#> 28:        10      G      B 16.209695    B-G        3
#> 29:        11      E      H 16.730634    E-H        5
#> 30:        11      H      E 16.730634    E-H        5
#> 31:        12      E      H 25.731123    E-H        5
#> 32:        12      H      E 25.731123    E-H        5
#> 33:        13      E      H 21.746311    E-H        5
#> 34:        13      H      E 21.746311    E-H        5
#>     timegroup    ID1    ID2  distance dyadID fusionID
#>         <int> <char> <char>     <num> <char>    <int>
```
