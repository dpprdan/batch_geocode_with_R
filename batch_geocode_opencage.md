Batch geocoding with opencage
================
Daniel Possenriede
2018-02-12

... and some other ideas & questions regarding opencage

Motivation
==========

I'd like to use `opencage` for batch (forward) geocoding, usually a few hundred addresses at the time. Most of the time the addresses are in a dataframe together with other information (so that's why I am using the `places_df`).

``` r
suppressPackageStartupMessages(library(tidyverse))
library(opencage)

elphi <- "Elbphilharmonie, Hamburg"
sherlock <- "Baker Street 221b, London"
triumph <- "Arc de Triomphe de l’Etoile, Paris"

places <- 
  c(elphi, 
    sherlock,
    triumph)

places_df <- data.frame(places, stringsAsFactors = FALSE)
```

`opencage_forward` is not vectorized, so giving it more than one address results in an error

``` r
opencage_forward(places)
#> Error in vapply(elements, encode, character(1)): values must be length 1,
#>  but FUN(X[[1]]) result is length 3
```

(maybe it could provide a better error message?)

But one ["just"](https://github.com/ropensci/opencage/issues/19) needs to use `purrr`.

Purrrify
========

I came up with two different, though similar ways to do this, but there probably is another/better way?

Either `map` `opencage_forward` over the placename column, turn the results into a dataframe and join that dataframe with the original dataframe (you need `add_request` to be `TRUE` for that).

``` r
oc_places_lst <- purrr::map(places_df[["places"]], opencage_forward, limit = 1)
oc_places_df <- suppressWarnings(purrr::map_df(oc_places_lst, "results"))
oc_places_df <- left_join(places_df, oc_places_df, by = c("places" = "query"))
```

(The `suppressWarning` is due to a lot of warnings about unequal factor levels. Maybe add a `stringsAsFactors = FALSE` to `opencage_parse`?)

Or `mutate` the result from a `map`ed `opencage_forward` into a list column

``` r

oc_places_df2 <- 
  places_df %>% 
  mutate(
    oc = purrr::map(.[["places"]], opencage_forward, limit = 1) %>% 
      purrr::map("results")
  )
oc_places_df2 <- suppressWarnings(unnest(oc_places_df2, oc))
```

Problems with this approach
---------------------------

I find both not very satisfactory, because they are both not very user-friendly IMHO and easily fail with unusual data.

For example if one gets a `NULL` result

``` r
not_a_place <- "dpprdan"

not_all_places <- c(places, not_a_place)

not_all_places_df <- tibble(not_all_places)

oc_na_places_df2 <- 
  not_all_places_df %>% 
  mutate(
    oc = purrr::map(.[["not_all_places"]], opencage_forward, limit = 1) %>% 
      purrr::map("results")
  )

oc_na_places_df2 <- suppressWarnings(unnest(oc_na_places_df2, oc))
#> Error: Each column must either be a list of vectors or a list of data frames [oc]
```

There probably is a way for `unnest` to deal with NULL lists, but I am not aware of it. `NULL` results do not seem to be a problem with the first method, though.

There will be unexpected results with the first method if one is not carefull, though, when there are address duplicates in the list.

``` r
places_dubl <- 
  c(places, elphi)

places_dubl_df <- data.frame(places_dubl, stringsAsFactors = FALSE)

oc_places_dubl_lst <-
  purrr::map(places_dubl_df[["places_dubl"]], opencage_forward, limit = 1)
  
oc_places_dubl_df <-
  suppressWarnings(purrr::map_df(oc_places_dubl_lst, "results"))
oc_places_dubl_df <-
  left_join(
    places_dubl_df,
    oc_places_dubl_df,
    by = c("places_dubl" = "query")
  )
  
nrow(places_dubl_df) == nrow(oc_places_dubl_df)
#> [1] FALSE
```

This can be avoided if by doing a `unique(oc_places_dubl_df)` before the `left_join`.

Some other ideas for improvement(?)
===================================

url-only
--------

For debugging I find an `urlonly` option quite handy. What does the url, the package sends to the API, look like and is it really what I think it is? See e.g. `ggmap::geocode`:

``` r
ggmap::geocode("1600 pennsylvania avenue, washington dc", urlonly = TRUE)
#> [1] "https://maps.googleapis.com/maps/api/geocode/json?address=1600%20pennsylvania%20avenue%2C%20washington%20dc"
```

Better (HTTP) error-handling
----------------------------

HTTP errors result in an R error (as opposed to a warning) and stop the whole function. Now, when a function errors out, e.g. because you loose the internet connection during a batch geocode, all results until then will be lost. I don't know how to fake a shaky internet connection at the moment, so as an example see this with an invalid key.

``` r
# catch errors in column
opencage_forward(elphi, key = "not_a_valid_key")
#> Error: HTTP failure: 403
#> Invalid or missing api key (forbidden)
```

It would be better to get a warning (maybe written into a separate column (purrr::possibly or something from the attempt package)) than to get an error.

Rate-Limiting
-------------

`opencage_forward` and `opencage_reverse`are not rate-limited (right?), i.e. there is no limit on how often they call the API per second. However, dependend on the type of [subscription](https://geocoder.opencagedata.com/pricing). opencage (the API) allows between 1 and 15 requests per second. This can become a problem with batch geocoding and even if it does not, it is simply nice to obey the rate limits. So maybe use `ratelimitr` and make the `rate` adjustable via `options()`, e.g. `limit_rate(opencage_get, getOption(OPENCAGE.rate)`). Regarding options, I like the approach of ggmap (v. 2.7), with all ggmap options grouped in a [list](https://github.com/dkahle/ggmap/blob/master/R/zzz.R) that can be [set](https://github.com/dkahle/ggmap/blob/master/R/register_google.R) and [read](https://github.com/dkahle/ggmap/blob/master/R/ggmap_credentials.R) with custom functions.

Encoding
--------

I thought I had an issue once with non-cp1252 ("latin-1" in R) strings. But apparently this works.

``` r
palace <- "Дворцовая пл., Санкт-Петербург, Россия"
opencage_forward(palace)
#> $results
#> # A tibble: 1 x 51
#>   annotations.DMS.~ annotations.DMS.~ annotations.ITM.e~ annotations.ITM.~
#>   <fct>             <fct>             <fct>              <fct>            
#> 1 54° 21' 39.99600~ 7° 25' 0.01200''~ 637913.457         845984.333       
#> # ... with 47 more variables: annotations.MGRS <fct>,
#> #   annotations.Maidenhead <fct>, annotations.Mercator.x <fct>,
#> #   annotations.Mercator.y <fct>, annotations.OSGB.easting <fct>,
#> #   annotations.OSGB.gridref <fct>, annotations.OSGB.northing <fct>,
#> #   annotations.OSM.url <fct>, annotations.callingcode <fct>,
#> #   annotations.currency.decimal_mark <fct>,
#> #   annotations.currency.html_entity <fct>,
#> #   annotations.currency.iso_code <fct>,
#> #   annotations.currency.iso_numeric <fct>,
#> #   annotations.currency.name <fct>,
#> #   annotations.currency.smallest_denomination <fct>,
#> #   annotations.currency.subunit <fct>,
#> #   annotations.currency.subunit_to_unit <fct>,
#> #   annotations.currency.symbol <fct>,
#> #   annotations.currency.symbol_first <fct>,
#> #   annotations.currency.thousands_separator <fct>,
#> #   annotations.flag <fct>, annotations.geohash <fct>,
#> #   annotations.qibla <fct>, annotations.sun.rise.apparent <fct>,
#> #   annotations.sun.rise.astronomical <fct>,
#> #   annotations.sun.rise.civil <fct>, annotations.sun.rise.nautical <fct>,
#> #   annotations.sun.set.apparent <fct>,
#> #   annotations.sun.set.astronomical <fct>,
#> #   annotations.sun.set.civil <fct>, annotations.sun.set.nautical <fct>,
#> #   annotations.timezone.name <fct>,
#> #   annotations.timezone.now_in_dst <fct>,
#> #   annotations.timezone.offset_sec <fct>,
#> #   annotations.timezone.offset_string <fct>,
#> #   annotations.timezone.short_name <fct>,
#> #   annotations.what3words.words <fct>,
#> #   `components.ISO_3166-1_alpha-2` <fct>, components._type <fct>,
#> #   components.country <fct>, components.country_code <fct>,
#> #   components.state <fct>, confidence <fct>, formatted <fct>,
#> #   geometry.lat <dbl>, geometry.lng <dbl>, query <chr>
#> 
#> $total_results
#> [1] 1
#> 
#> $time_stamp
#> [1] "2018-02-12 11:08:43 CET"
#> 
#> $rate_info
#> # A tibble: 1 x 3
#>   limit remaining reset              
#>   <int>     <int> <dttm>             
#> 1  2500      2475 2018-02-13 01:00:00
```

Multiple countrycodes
---------------------

opencage supports \[multiple country codes(<https://geocoder.opencagedata.com/api#forward-opt>), but `opencage` does not (yet).

``` r
# French territories according to the opencage website
french_territories <- 
  c("fr", "bl", "gf", "gp", "mf", "mq", "nc", "pf", "pm", "re", "tf", "wf", "yt") %>% 
  toupper()

opencage_forward("Arc de Triomphe de l’Etoile, Paris", countrycode = french_territories)
#> Warning in if (!(countrycode %in% countrycodes$Code)) {: the condition has
#> length > 1 and only the first element will be used
#> Error in vapply(elements, encode, character(1)): values must be length 1,
#>  but FUN(X[[2]]) result is length 13
```
