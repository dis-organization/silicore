
<!-- README.md is generated from README.Rmd. Please edit that file -->
Overview
========

This is a super stripped down core for silicate. All it has is the SC0 model, which is

-   `coord`, all coordinates from the input
-   `segment`, the .vx0 and .vx1 index of every input segment (row of coord)
-   `geometry` map (the record of any paths, `cumsum(nrow)` is a count-index into coord)
-   `data` (the feature table)

Compare with silicate:

``` r
library(silicate)
library(silicore)
str(sc_coord(minimal_mesh))
#> Classes 'tbl_df', 'tbl' and 'data.frame':    19 obs. of  2 variables:
#>  $ x_: num  0 0 0.75 1 0.5 0.8 0.69 0 0.2 0.5 ...
#>  $ y_: num  0 1 1 0.8 0.7 0.6 0 0 0.2 0.2 ...
str(SC0(minimal_mesh)$coord) ## exactly the same
#> Classes 'tbl_df', 'tbl' and 'data.frame':    19 obs. of  2 variables:
#>  $ x_: num  0 0 0.75 1 0.5 0.8 0.69 0 0.2 0.5 ...
#>  $ y_: num  0 1 1 0.8 0.7 0.6 0 0 0.2 0.2 ...
```

``` r
sc_edge(minimal_mesh)     ## relational labels
#> # A tibble: 15 x 3
#>    .vertex0   .vertex1   edge_     
#>    <chr>      <chr>      <chr>     
#>  1 713af8251a 5035049e1a dbcf37ddef
#>  2 5035049e1a 397b6759c9 8f0741fb5c
#>  3 397b6759c9 fa3f42c123 b33e91bdb1
#>  4 fa3f42c123 03e8b4f99c 1946d339d5
#>  5 03e8b4f99c 21d4bb7a89 8764404069
#>  6 21d4bb7a89 b5ea053b4c 4670000e0a
#>  7 b5ea053b4c 713af8251a 9d8a9239a4
#>  8 ffa1a29f95 f508b128c6 2a1cecbeb2
#>  9 f508b128c6 a7baf49eda 59be468d05
#> 10 a7baf49eda fe660e2bc2 138ecc20a1
#> 11 fe660e2bc2 ef243cad39 e4b4a88555
#> 12 ef243cad39 ffa1a29f95 f7645063ea
#> 13 21d4bb7a89 cc4a991b6d e29beb0274
#> 14 cc4a991b6d b63bbfddf8 39ac1b1cdb
#> 15 b63bbfddf8 b5ea053b4c 1584e2f3f6
SC0(minimal_mesh)$segment ## purely structure index
#> # A tibble: 16 x 2
#>     .vx0  .vx1
#>    <int> <int>
#>  1     1     2
#>  2     2     3
#>  3     3     4
#>  4     4     5
#>  5     5     6
#>  6     6     7
#>  7     7     8
#>  8     9    10
#>  9    10    11
#> 10    11    12
#> 11    12    13
#> 12    13    14
#> 13    15    16
#> 14    16    17
#> 15    17    18
#> 16    18    19
```

``` r
sc_path(minimal_mesh)      
#> # A tibble: 3 x 7
#>    ncol type         subobject object object_    path_      ncoords_
#>   <int> <chr>            <int>  <int> <chr>      <chr>         <int>
#> 1     2 MULTIPOLYGON         1      1 4f70cac663 8d2dc8e933        8
#> 2     2 MULTIPOLYGON         1      1 4f70cac663 e5ef0fda56        6
#> 3     2 MULTIPOLYGON         1      2 a806eff7b8 62af015982        5
SC0(minimal_mesh)$geometry ## no relational labels
#> # A tibble: 3 x 6
#>    nrow  ncol type         subobject object  path
#>   <int> <int> <chr>            <int>  <int> <int>
#> 1     8     2 MULTIPOLYGON         1      1     1
#> 2     6     2 MULTIPOLYGON         1      1     2
#> 3     5     2 MULTIPOLYGON         1      2     3
```

``` r
sc_object(minimal_mesh)  
#> # A tibble: 2 x 1
#>       a
#> * <int>
#> 1     1
#> 2     2
SC0(minimal_mesh)$data  ## the same, geometry$object is the row number
#> # A tibble: 2 x 1
#>       a
#> * <int>
#> 1     1
#> 2     2
```

Performance is good.

``` r
rbenchmark::benchmark(SC0(minimal_mesh), 
                      SC(minimal_mesh))
#>                test replications elapsed relative user.self sys.self
#> 2  SC(minimal_mesh)          100   2.047    1.555     2.039    0.008
#> 1 SC0(minimal_mesh)          100   1.316    1.000     1.309    0.008
#>   user.child sys.child
#> 2          0         0
#> 1          0         0
```

``` r
rbenchmark::benchmark(SC0(inlandwaters), 
                      SC(inlandwaters), replications = 10)
#>                test replications elapsed relative user.self sys.self
#> 2  SC(inlandwaters)           10  15.185   34.047    15.157    0.028
#> 1 SC0(inlandwaters)           10   0.446    1.000     0.443    0.004
#>   user.child sys.child
#> 2          0         0
#> 1          0         0
```

For now we are ignoring

-   relational indexes
-   geometric normalization (might be x/y, x/y/z, other spaces ...)
-   any explicitly sequential storage, i.e. ARC and PATH probably will be rebuilt completely

We import the internal `silicate:::get_projection.sf` and for now rely on `gibble` to give the mapping between coordinates and paths.

This is being tried because silicate was originally defined around PATH, and even SC was defined in terms of it - unnecessarily. Once [we tried](http://rpubs.com/cyclemumner/367272) a purely tidyverse approach to the problem of decomposing paths to edges it seemed natural to use that as a basis. It means that we can keep the run-length index (gibble) as a way of storing higher level groupings, including holes and multipolygons *optionally*. Here we don't need to encode the actual sequence of coordinates along paths or arcs, because i) it's implicit in geometry/coord tables prior to vertex de-duplication and ii) we might discard them favouring edge-traversal as a way of reconstructing sequences.

Can we get sense out of it?

``` r
x <- SC0(minimal_mesh)
library(ggplot2)
library(dplyr)
#> 
#> Attaching package: 'dplyr'
#> The following objects are masked from 'package:stats':
#> 
#>     filter, lag
#> The following objects are masked from 'package:base':
#> 
#>     intersect, setdiff, setequal, union
tab <- tibble(xs = x$coord$x_[x$segment$.vx0], ys = x$coord$y_[x$segment$.vx0], 
              xend = x$coord$x_[x$segment$.vx1], yend = x$coord$y_[x$segment$.vx1])
ggplot(tab, aes(xs, ys, xend  =xend, yend = yend)) + geom_segment()
```

![](README-unnamed-chunk-8-1.png)

``` r

x <- SC0(inlandwaters)
## the  sixth object
tst <- rep(x$geometry$object, x$geometry$nrow) == 4
idx <- which(tst)
g <-  rep(x$geometry$path, x$geometry$nrow)
segment <- x$segment %>% dplyr::filter(.vx0 %in% idx & .vx1 %in% idx)
tab <- tibble(xs = x$coord$x_[segment$.vx0], ys = x$coord$y_[segment$.vx0], 
              xend = x$coord$x_[segment$.vx1], yend = x$coord$y_[segment$.vx1], 
              path = g[segment$.vx0])
ggplot(tab, aes(xs, ys, xend  =xend, yend = yend, colour = factor(path))) + geom_segment() + guides(colour = FALSE)
```

![](README-unnamed-chunk-8-2.png)

``` r


x <- SC0(dodgr::hampi)
g <-  rep(x$geometry$path, x$geometry$nrow)
tab <- tibble(xs = x$coord$x_[x$segment$.vx0], ys = x$coord$y_[x$segment$.vx0], 
              xend = x$coord$x_[x$segment$.vx1], yend = x$coord$y_[x$segment$.vx1], 
              path = g[x$segment$.vx0])
ggplot(tab, aes(xs, ys, xend  =xend, yend = yend, colour = path)) + geom_segment()
```

![](README-unnamed-chunk-8-3.png)

If anyone can come up with a better name than `gibble` or `geometry` or `geometry map` for that thing, I'll be really grateful.
