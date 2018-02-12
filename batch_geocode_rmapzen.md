Batch geocoding with rmapzen
================
Daniel Possenriede
2018-02-12

<!-- bath_geocode_rmapzen.md is generated from bath_geocode_rmapzen.Rmd Please edit that file -->
``` r
suppressPackageStartupMessages(library(here))
suppressPackageStartupMessages(library(tidyverse))
library(rmapzen)

elphi <- "Elbphilharmonie, Hamburg"
sherlock <- "Baker Street 221b, London"
triumph <- "Arc de Triomphe de l’Etoile, Paris"

places <- 
  c(elphi, 
    sherlock,
    triumph)

places_df <- data.frame(places, stringsAsFactors = FALSE)
```

`mz_search` is the general function for forward geocoding. It creates an `mapzen_geo_list` object (which has it's own `print` method).

``` r
(mz_search_elphi <- mz_search(elphi))
#> GeoJSON response from Mapzen
#> Attribution info: http://pelias.mapzen.com/v1/attribution 
#> Bounds (lon/lat): (9.98, 53.54) - (9.99, 53.54)
#> 9 locations:
#>    Elbphilharmonie (9.99, 53.54)
#>    Elbphilharmonie (9.98, 53.54)
#>    Baumwall (Elbphilharmonie) (9.98, 53.54)
#>    Elbphilharmonie Besucherzentrum (9.99, 53.54)
#>    Parkhaus Elbphilharmonie (9.98, 53.54)
#>   ...
# saveRDS(mz_search_elphi, here("rmapzen/mz_search_elphi.rds"))
```

The results are relatively "raw", i.e. they have only been parsed by `jsonlite::fromJSON()`.

`mz_geocode` is a dedicated function to geocode an address, utilizing the more general `mz_search` function. It returns a data.frame with a formated address, longitude, latitude and confidence.

``` r
(mz_geocode_elphi <- mz_geocode(elphi))
#> # A tibble: 1 x 4
#>   geocode_address       geocode_longitu~ geocode_latitude geocode_confide~
#> * <chr>                            <dbl>            <dbl>            <dbl>
#> 1 Elbphilharmonie, Ham~             9.99             53.5            0.945
# saveRDS(mz_geocode_elphi, here("rmapzen/mz_geocode_elphi.rds"))
```

Both are not vectorized, so this gives an error.

``` r
mz_geocode(places)
#> Error: text is not a string (a length one character vector).
```

Utilizing `purrr::map_df`

``` r

(mz_bind_places_df <- 
  places_df[["places"]] %>% 
  purrr::map_df(mz_geocode) %>% 
  bind_cols(places_df, .))
#>                               places                   geocode_address
#> 1           Elbphilharmonie, Hamburg Elbphilharmonie, Hamburg, Germany
#> 2          Baker Street 221b, London   London, England, United Kingdom
#> 3 Arc de Triomphe de l’Etoile, Paris    Arc De Triomphe, Paris, France
#>   geocode_longitude geocode_latitude geocode_confidence
#> 1          9.986492         53.54031              0.945
#> 2         -0.099076         51.50965              0.600
#> 3          2.293700         48.87674              0.500
```

Or `mutate` the result from a `map`ed `mz_geocode` into a list column

``` r

(mz_mutate_places_df <- 
  places_df %>% 
  mutate(codes = map(places, mz_geocode)) %>% 
  unnest(codes))
#>                               places                   geocode_address
#> 1           Elbphilharmonie, Hamburg Elbphilharmonie, Hamburg, Germany
#> 2          Baker Street 221b, London   London, England, United Kingdom
#> 3 Arc de Triomphe de l’Etoile, Paris    Arc De Triomphe, Paris, France
#>   geocode_longitude geocode_latitude geocode_confidence
#> 1          9.986492         53.54031              0.945
#> 2         -0.099076         51.50965              0.600
#> 3          2.293700         48.87674              0.500
```

Problems
--------

### `NULL` results

For example if one gets a `NULL` result

``` r
not_a_place <- "dpprdan"

not_all_places <- c(not_a_place, places)

not_all_places_df <- tibble(not_all_places)

mz_na_places_df2 <- 
  not_all_places_df %>% 
  mutate(codes = map(places, mz_geocode)) %>% 
  unnest(codes)
#> Error in mutate_impl(.data, dots): Column `codes` must be length 4 (the number of rows) or one, not 3
```

There probably is a way for `unnest` to deal with NULL lists, but I am not aware of it. This does work with the first method.

### Encoding

Apparently not a problem

``` r
palace <- "Дворцовая пл., Санкт-Петербург, Россия"

mz_geocode(palace)
#> Error in mz_geocode(palace): Tried to geocode <U+0414><U+0432><U+043E><U+0440><U+0446><U+043E><U+0432><U+0430><U+044F> <U+043F><U+043B>., <U+0421><U+0430><U+043D><U+043A><U+0442>-<U+041F><U+0435><U+0442><U+0435><U+0440><U+0431><U+0443><U+0440><U+0433>, <U+0420><U+043E><U+0441><U+0441><U+0438><U+044F> but there were no results
```

### Getting more information

It is not really easy to extract additional information from mz\_search result. Compare for example `ggmap::geocode` output options, `c("latlon", "latlona", "more", "all")`, which returns dataframes with varying information (`"all"` returns a list, though). A option/function that outputs all `feature`s as a data.frame would suffice, though. But there is already an [issue](https://github.com/tarakc02/rmapzen/issues/12) for this.

``` r
mz_search_elphi[["features"]][[1]]
#> $type
#> [1] "Feature"
#> 
#> $geometry
#> $geometry$type
#> [1] "Point"
#> 
#> $geometry$coordinates
#> $geometry$coordinates[[1]]
#> [1] 9.986492
#> 
#> $geometry$coordinates[[2]]
#> [1] 53.54031
#> 
#> 
#> 
#> $properties
#> $properties$id
#> [1] "way:376083169"
#> 
#> $properties$gid
#> [1] "openstreetmap:venue:way:376083169"
#> 
#> $properties$layer
#> [1] "venue"
#> 
#> $properties$source
#> [1] "openstreetmap"
#> 
#> $properties$source_id
#> [1] "way:376083169"
#> 
#> $properties$name
#> [1] "Elbphilharmonie"
#> 
#> $properties$confidence
#> [1] 0.945
#> 
#> $properties$accuracy
#> [1] "point"
#> 
#> $properties$country
#> [1] "Germany"
#> 
#> $properties$country_gid
#> [1] "whosonfirst:country:85633111"
#> 
#> $properties$country_a
#> [1] "DEU"
#> 
#> $properties$region
#> [1] "Hamburg"
#> 
#> $properties$region_gid
#> [1] "whosonfirst:region:85682505"
#> 
#> $properties$region_a
#> [1] "HH"
#> 
#> $properties$county
#> [1] "Hamburg "
#> 
#> $properties$county_gid
#> [1] "whosonfirst:county:102064053"
#> 
#> $properties$locality
#> [1] "Hamburg"
#> 
#> $properties$locality_gid
#> [1] "whosonfirst:locality:101748841"
#> 
#> $properties$locality_a
#> [1] "HH"
#> 
#> $properties$neighbourhood
#> [1] "Kleiner Grasbrook"
#> 
#> $properties$neighbourhood_gid
#> [1] "whosonfirst:neighbourhood:85899367"
#> 
#> $properties$continent
#> [1] "Europe"
#> 
#> $properties$continent_gid
#> [1] "whosonfirst:continent:102191581"
#> 
#> $properties$label
#> [1] "Elbphilharmonie, Hamburg, Germany"
#> 
#> 
#> $bbox
#> $bbox[[1]]
#> [1] 9.986186
#> 
#> $bbox[[2]]
#> [1] 53.54013
#> 
#> $bbox[[3]]
#> [1] 9.987036
#> 
#> $bbox[[4]]
#> [1] 53.54042
```
