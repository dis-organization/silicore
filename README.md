
<!-- README.md is generated from README.Rmd. Please edit that file -->
Overview
========

sc0 is a super stripped down core for silicate. All it has is the SC0 model, which is

-   `coord`, all coordinates from the input
-   `segment`, the .vx0 and .vx1 index of every input segment (row of coord)
-   `geometry` map (the record of any paths, `cumsum(nrow)` is a count-index into coord)
-   `data` (the feature table)

Compare with silicate:

``` r
library(silicate)
library(silicore)
sc_coord(minimal_mesh)
#> # A tibble: 19 x 2
#>       x_    y_
#>    <dbl> <dbl>
#>  1 0.    0.   
#>  2 0.    1.00 
#>  3 0.750 1.00 
#>  4 1.00  0.800
#>  5 0.500 0.700
#>  6 0.800 0.600
#>  7 0.690 0.   
#>  8 0.    0.   
#>  9 0.200 0.200
#> 10 0.500 0.200
#> 11 0.500 0.400
#> 12 0.300 0.600
#> 13 0.200 0.400
#> 14 0.200 0.200
#> 15 0.690 0.   
#> 16 0.800 0.600
#> 17 1.10  0.630
#> 18 1.23  0.300
#> 19 0.690 0.
SC0(minimal_mesh)$coord
#> # A tibble: 19 x 2
#>       x_    y_
#>    <dbl> <dbl>
#>  1 0.    0.   
#>  2 0.    1.00 
#>  3 0.750 1.00 
#>  4 1.00  0.800
#>  5 0.500 0.700
#>  6 0.800 0.600
#>  7 0.690 0.   
#>  8 0.    0.   
#>  9 0.200 0.200
#> 10 0.500 0.200
#> 11 0.500 0.400
#> 12 0.300 0.600
#> 13 0.200 0.400
#> 14 0.200 0.200
#> 15 0.690 0.   
#> 16 0.800 0.600
#> 17 1.10  0.630
#> 18 1.23  0.300
#> 19 0.690 0.
```

``` r
sc_edge(minimal_mesh)
#> # A tibble: 15 x 3
#>    .vertex0   .vertex1   edge_     
#>    <chr>      <chr>      <chr>     
#>  1 9de16738b2 e1a291337b fbf2da3c6c
#>  2 e1a291337b 0d2483341b 0a1278d394
#>  3 0d2483341b 3ca4f5ca4c 8ce1cad3ff
#>  4 3ca4f5ca4c a156f3a93c 42a087d205
#>  5 a156f3a93c 11ce50bc32 b590566b8c
#>  6 11ce50bc32 eb3f7e8d38 ca44e65c6d
#>  7 eb3f7e8d38 9de16738b2 6ecb6a62e3
#>  8 e61045f605 2a63eb9bcf 24bf4950b2
#>  9 2a63eb9bcf a347e613ea 9aaeefe540
#> 10 a347e613ea b5172c424b 36b5b97b37
#> 11 b5172c424b 16322f7d93 a653da0246
#> 12 16322f7d93 e61045f605 46168ca83b
#> 13 11ce50bc32 dc968a70c9 c10227cc14
#> 14 dc968a70c9 5057e93340 a9bb38610b
#> 15 5057e93340 eb3f7e8d38 879ab5b953
SC0(minimal_mesh)$segment
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
#> 1     2 MULTIPOLYGON         1      1 532e1dbcd1 8eed5817f4        8
#> 2     2 MULTIPOLYGON         1      1 532e1dbcd1 929fc38d22        6
#> 3     2 MULTIPOLYGON         1      2 24f79d6091 a5459330a8        5
SC0(minimal_mesh)$geometry
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
SC0(minimal_mesh)$data
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
#> 2  SC(minimal_mesh)          100   2.045    1.561     2.044     0.00
#> 1 SC0(minimal_mesh)          100   1.310    1.000     1.269     0.04
#>   user.child sys.child
#> 2          0         0
#> 1          0         0
```

``` r
rbenchmark::benchmark(SC0(inlandwaters), 
                      SC(inlandwaters), replications = 10)
#>                test replications elapsed relative user.self sys.self
#> 2  SC(inlandwaters)           10  15.005   32.478    14.948    0.052
#> 1 SC0(inlandwaters)           10   0.462    1.000     0.458    0.004
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
