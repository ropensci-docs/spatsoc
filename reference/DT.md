# Movement of 10 "Newfoundland Bog Cows"

A dataset containing the GPS relocations of 10 individuals in winter
2016-2017.

## Format

A data.table with 14297 rows and 5 variables:

- ID:

  individual identifier

- X:

  X coordinate of the relocation (UTM 36N)

- Y:

  Y coordinate of the relocation (UTM 36N)

- datetime:

  character string representing the date time

- population:

  sub population within the individuals

## Examples

``` r
# Load data.table
library(data.table)
#> 
#> Attaching package: ‘data.table’
#> The following object is masked from ‘package:base’:
#> 
#>     %notin%

# Read example data
DT <- fread(system.file("extdata", "DT.csv", package = "spatsoc"))
```
