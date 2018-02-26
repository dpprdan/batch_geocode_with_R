Some Tests of the Opencage Package
================
Daniel Possenriede (@dpprdan)
2018-02-26

``` r
# devtools::install_github("dpprdan/opencage@output_options")
suppressPackageStartupMessages(library(tidyverse))
library(opencage)

elphi <- "Elbphilharmonie, Hamburg"
sherlock <- "Baker Street 221b, London"
triumph <- "Arc de Triomphe de l’Etoile, Paris"

places <- 
  c(elphi, 
    sherlock,
    triumph)

places_df <- tibble(places) %>% rowid_to_column()
```

Single placename
================

return dataframe, multiple results
----------------------------------

``` r
oc_forward(triumph, output = "df_list") # df_list is default
#> # A tibble: 2 x 62
#>   confidence formatted   MGRS   Maidenhead callingcode flag  geohash qibla
#> *      <int> <chr>       <chr>  <chr>            <int> <chr> <chr>   <dbl>
#> 1          9 Arc de Tri~ 31UDQ~ JN18du59jq          33 "\U0~ u09wh1~  119.
#> 2          6 Paris, Fra~ 31UDQ~ JN18eu14ut          33 "\U0~ u09tvm~  119.
#> # ... with 54 more variables: wikidata <chr>, DMS_lat <chr>,
#> #   DMS_lng <chr>, Mercator_x <dbl>, Mercator_y <dbl>, OSM_edit_url <chr>,
#> #   OSM_url <chr>, currency_alternate_symbols <list>,
#> #   currency_decimal_mark <chr>, currency_html_entity <chr>,
#> #   currency_iso_code <chr>, currency_iso_numeric <int>,
#> #   currency_name <chr>, currency_smallest_denomination <int>,
#> #   currency_subunit <chr>, currency_subunit_to_unit <int>,
#> #   currency_symbol <chr>, currency_symbol_first <int>,
#> #   currency_thousands_separator <chr>, sun_rise_apparent <int>,
#> #   sun_rise_astronomical <int>, sun_rise_civil <int>,
#> #   sun_rise_nautical <int>, sun_set_apparent <int>,
#> #   sun_set_astronomical <int>, sun_set_civil <int>,
#> #   sun_set_nautical <int>, timezone_name <chr>,
#> #   timezone_now_in_dst <int>, timezone_offset_sec <int>,
#> #   timezone_offset_string <int>, timezone_short_name <chr>,
#> #   what3words_words <chr>, northeast_lat <dbl>, northeast_lng <dbl>,
#> #   southwest_lat <dbl>, southwest_lng <dbl>, ISO_3166_1_alpha_2 <chr>,
#> #   type <chr>, attraction <chr>, city <chr>, city_district <chr>,
#> #   country <chr>, country_code <chr>, county <chr>, pedestrian <chr>,
#> #   political_union <chr>, postcode <chr>, state <chr>, suburb <chr>,
#> #   local_administrative_area <chr>, town <chr>, lat <dbl>, lng <dbl>
```

return list
-----------

Output not shown, because output contains API key (in `$request$key`)

``` r
oc_forward(triumph, output = "json_list")
```

return JSON string
------------------

E.g. for use with `jqr`, output not shown.

``` r
oc_forward(triumph, output = "json_list") %>% 
  jsonlite::toJSON()
```

Multiple results with column binding
====================================

return dataframe, single result
-------------------------------

``` r

places_df %>% 
  mutate(
    oc = oc_forward(places, limit = 1, output = "df_list")
  ) %>% 
  tidyr::unnest(oc)
#> # A tibble: 3 x 70
#>   rowid places  confidence formatted    MGRS  Maidenhead callingcode flag 
#>   <int> <chr>        <int> <chr>        <chr> <chr>            <int> <chr>
#> 1     1 Elbphi~          9 Elbe Philha~ 32UN~ JO43xm89cv          49 "\U0~
#> 2     2 Baker ~          9 Sherlock Ho~ 30UX~ IO91wm05xq          44 "\U0~
#> 3     3 Arc de~          9 Arc de Trio~ 31UD~ JN18du59jq          33 "\U0~
#> # ... with 62 more variables: geohash <chr>, qibla <dbl>, wikidata <chr>,
#> #   DMS_lat <chr>, DMS_lng <chr>, Mercator_x <dbl>, Mercator_y <dbl>,
#> #   OSM_edit_url <chr>, OSM_url <chr>, currency_alternate_symbols <list>,
#> #   currency_decimal_mark <chr>, currency_html_entity <chr>,
#> #   currency_iso_code <chr>, currency_iso_numeric <int>,
#> #   currency_name <chr>, currency_smallest_denomination <int>,
#> #   currency_subunit <chr>, currency_subunit_to_unit <int>,
#> #   currency_symbol <chr>, currency_symbol_first <int>,
#> #   currency_thousands_separator <chr>, sun_rise_apparent <int>,
#> #   sun_rise_astronomical <int>, sun_rise_civil <int>,
#> #   sun_rise_nautical <int>, sun_set_apparent <int>,
#> #   sun_set_astronomical <int>, sun_set_civil <int>,
#> #   sun_set_nautical <int>, timezone_name <chr>,
#> #   timezone_now_in_dst <int>, timezone_offset_sec <int>,
#> #   timezone_offset_string <int>, timezone_short_name <chr>,
#> #   what3words_words <chr>, northeast_lat <dbl>, northeast_lng <dbl>,
#> #   southwest_lat <dbl>, southwest_lng <dbl>, ISO_3166_1_alpha_2 <chr>,
#> #   type <chr>, arts_centre <chr>, city_district <chr>, country <chr>,
#> #   country_code <chr>, house_number <chr>, pedestrian <chr>,
#> #   political_union <chr>, postcode <chr>, state <chr>, suburb <chr>,
#> #   lat <dbl>, lng <dbl>, OSGB_easting <dbl>, OSGB_gridref <chr>,
#> #   OSGB_northing <dbl>, city <chr>, museum <chr>, road <chr>,
#> #   state_district <chr>, attraction <chr>, county <chr>
```

return dataframe, multiple results, no\_annotations
---------------------------------------------------

``` r

places_df %>% 
  mutate(
    oc = oc_forward(places, output = "df_list", no_annotations = TRUE)) %>% 
  tidyr::unnest(oc)
#> # A tibble: 8 x 30
#>   rowid places     confidence formatted        northeast_lat northeast_lng
#>   <int> <chr>           <int> <chr>                    <dbl>         <dbl>
#> 1     1 Elbphilha~          9 Elbe Philharmon~          53.5         9.98 
#> 2     1 Elbphilha~          3 Hamburg, Germany          53.7        10.2  
#> 3     2 Baker Str~          9 Sherlock Holmes~          51.5        -0.158
#> 4     2 Baker Str~         10 221B Baker Stre~          51.5        -0.158
#> 5     2 Baker Str~          1 London, Greater~          51.7         0.336
#> 6     2 Baker Str~         10 Baker Street, L~          NA          NA    
#> 7     3 Arc de Tr~          9 Arc de Triomphe~          48.9         2.30 
#> 8     3 Arc de Tr~          6 Paris, France             48.9         2.42 
#> # ... with 24 more variables: southwest_lat <dbl>, southwest_lng <dbl>,
#> #   ISO_3166_1_alpha_2 <chr>, type <chr>, arts_centre <chr>,
#> #   city_district <chr>, country <chr>, country_code <chr>,
#> #   house_number <chr>, pedestrian <chr>, political_union <chr>,
#> #   postcode <chr>, state <chr>, suburb <chr>, town <chr>, lat <dbl>,
#> #   lng <dbl>, city <chr>, museum <chr>, road <chr>, state_district <chr>,
#> #   county <chr>, attraction <chr>, local_administrative_area <chr>
```

dataframe with R list column with parsed JSON
---------------------------------------------

output not shown

``` r

places_df %>% 
  mutate(
    oc = oc_forward(places, limit = 1, output = "json_list"))
    
```

input character vector, output dataframe
----------------------------------------

``` r
oc_forward(places, limit = 1, output = "df_list") %>% 
  bind_rows()
#> # A tibble: 3 x 68
#>   confidence formatted   MGRS   Maidenhead callingcode flag  geohash qibla
#>        <int> <chr>       <chr>  <chr>            <int> <chr> <chr>   <dbl>
#> 1          9 Elbe Philh~ 32UNE~ JO43xm89cv          49 "\U0~ u1x0e4~  133.
#> 2          9 Sherlock H~ 30UXC~ IO91wm05xq          44 "\U0~ gcpvh7~  119.
#> 3          9 Arc de Tri~ 31UDQ~ JN18du59jq          33 "\U0~ u09wh1~  119.
#> # ... with 60 more variables: wikidata <chr>, DMS_lat <chr>,
#> #   DMS_lng <chr>, Mercator_x <dbl>, Mercator_y <dbl>, OSM_edit_url <chr>,
#> #   OSM_url <chr>, currency_alternate_symbols <list>,
#> #   currency_decimal_mark <chr>, currency_html_entity <chr>,
#> #   currency_iso_code <chr>, currency_iso_numeric <int>,
#> #   currency_name <chr>, currency_smallest_denomination <int>,
#> #   currency_subunit <chr>, currency_subunit_to_unit <int>,
#> #   currency_symbol <chr>, currency_symbol_first <int>,
#> #   currency_thousands_separator <chr>, sun_rise_apparent <int>,
#> #   sun_rise_astronomical <int>, sun_rise_civil <int>,
#> #   sun_rise_nautical <int>, sun_set_apparent <int>,
#> #   sun_set_astronomical <int>, sun_set_civil <int>,
#> #   sun_set_nautical <int>, timezone_name <chr>,
#> #   timezone_now_in_dst <int>, timezone_offset_sec <int>,
#> #   timezone_offset_string <int>, timezone_short_name <chr>,
#> #   what3words_words <chr>, northeast_lat <dbl>, northeast_lng <dbl>,
#> #   southwest_lat <dbl>, southwest_lng <dbl>, ISO_3166_1_alpha_2 <chr>,
#> #   type <chr>, arts_centre <chr>, city_district <chr>, country <chr>,
#> #   country_code <chr>, house_number <chr>, pedestrian <chr>,
#> #   political_union <chr>, postcode <chr>, state <chr>, suburb <chr>,
#> #   lat <dbl>, lng <dbl>, OSGB_easting <dbl>, OSGB_gridref <chr>,
#> #   OSGB_northing <dbl>, city <chr>, museum <chr>, road <chr>,
#> #   state_district <chr>, attraction <chr>, county <chr>
```

Source with problems
====================

Duplicates, empty placename, not a real placename (i.e. no result), NA placename.

``` r
places2 <- 
  c(elphi, 
    sherlock,
    triumph,
    elphi,
    "",
    "dpprdan")

places2_df <- 
  tibble(places2, stringsAsFactors = FALSE) %>%
  rowid_to_column()
```

``` r

places2_df %>% 
  mutate(
    oc = oc_forward(places2, output = "df_list", no_annotations = TRUE)) %>% 
  tidyr::unnest(oc)
#> # A tibble: 12 x 31
#>    rowid places2   stringsAsFactors confidence formatted     northeast_lat
#>    <int> <chr>     <lgl>                 <int> <chr>                 <dbl>
#>  1     1 Elbphilh~ FALSE                     9 Elbe Philhar~          53.5
#>  2     1 Elbphilh~ FALSE                     3 Hamburg, Ger~          53.7
#>  3     2 Baker St~ FALSE                     9 Sherlock Hol~          51.5
#>  4     2 Baker St~ FALSE                    10 221B Baker S~          51.5
#>  5     2 Baker St~ FALSE                     1 London, Grea~          51.7
#>  6     2 Baker St~ FALSE                    10 Baker Street~          NA  
#>  7     3 Arc de T~ FALSE                     9 Arc de Triom~          48.9
#>  8     3 Arc de T~ FALSE                     6 Paris, France          48.9
#>  9     4 Elbphilh~ FALSE                     9 Elbe Philhar~          53.5
#> 10     4 Elbphilh~ FALSE                     3 Hamburg, Ger~          53.7
#> 11     5 ""        FALSE                    NA <NA>                   NA  
#> 12     6 dpprdan   FALSE                    NA <NA>                   NA  
#> # ... with 25 more variables: northeast_lng <dbl>, southwest_lat <dbl>,
#> #   southwest_lng <dbl>, ISO_3166_1_alpha_2 <chr>, type <chr>,
#> #   arts_centre <chr>, city_district <chr>, country <chr>,
#> #   country_code <chr>, house_number <chr>, pedestrian <chr>,
#> #   political_union <chr>, postcode <chr>, state <chr>, suburb <chr>,
#> #   town <chr>, lat <dbl>, lng <dbl>, city <chr>, museum <chr>,
#> #   road <chr>, state_district <chr>, county <chr>, attraction <chr>,
#> #   local_administrative_area <chr>
```

Remaining Problems
==================

-   \[ \] countrycode per row not yet possible (need to change vectorisation in `oc_forward()` from `purrr::map` to `purrr::pmap`)
-   \[ \] `NA_character` as placename (returns results for `placename = "NA"`)
-   \[ \] `add_query` parameter (use placename directly from R, not via API roundtrip)
-   \[ \] handle (HTTP) errors while querying: `purrr::safely`?!

<details><summary>Session info</summary>

``` r
devtools::session_info()
#> Session info -------------------------------------------------------------
#>  setting  value                       
#>  version  R version 3.4.3 (2017-11-30)
#>  system   x86_64, mingw32             
#>  ui       RTerm                       
#>  language en                          
#>  collate  German_Germany.1252         
#>  tz       Europe/Berlin               
#>  date     2018-02-26
#> Packages -----------------------------------------------------------------
#>  package    * version    date       source                            
#>  assertthat   0.2.0      2017-04-11 CRAN (R 3.4.2)                    
#>  backports    1.1.2      2017-12-13 CRAN (R 3.4.3)                    
#>  base       * 3.4.3      2017-12-06 local                             
#>  bindr        0.1        2016-11-13 CRAN (R 3.4.0)                    
#>  bindrcpp   * 0.2        2017-06-17 CRAN (R 3.4.0)                    
#>  broom        0.4.3      2017-11-20 CRAN (R 3.4.2)                    
#>  cellranger   1.1.0      2016-07-27 CRAN (R 3.4.2)                    
#>  cli          1.0.0      2017-11-05 CRAN (R 3.4.2)                    
#>  colorspace   1.3-2      2016-12-14 CRAN (R 3.4.2)                    
#>  compiler     3.4.3      2017-12-06 local                             
#>  crayon       1.3.4      2018-02-12 Github (r-lib/crayon@95b3eae)     
#>  crul         0.5.2      2018-02-24 CRAN (R 3.4.3)                    
#>  curl         3.1        2017-12-12 CRAN (R 3.4.3)                    
#>  datasets   * 3.4.3      2017-12-06 local                             
#>  devtools     1.13.5     2018-02-18 CRAN (R 3.4.3)                    
#>  digest       0.6.15     2018-01-28 CRAN (R 3.4.3)                    
#>  dplyr      * 0.7.4      2017-09-28 CRAN (R 3.4.2)                    
#>  evaluate     0.10.1     2017-06-24 CRAN (R 3.4.0)                    
#>  forcats    * 0.3.0      2018-02-19 CRAN (R 3.4.3)                    
#>  foreign      0.8-69     2017-06-21 CRAN (R 3.4.0)                    
#>  ggplot2    * 2.2.1.9000 2017-09-07 Github (tidyverse/ggplot2@c592e32)
#>  glue         1.2.0.9000 2018-02-09 local                             
#>  graphics   * 3.4.3      2017-12-06 local                             
#>  grDevices  * 3.4.3      2017-12-06 local                             
#>  grid         3.4.3      2017-12-06 local                             
#>  gtable       0.2.0      2016-02-26 CRAN (R 3.4.2)                    
#>  haven        1.1.1      2018-01-18 CRAN (R 3.4.3)                    
#>  hms          0.4.1      2018-01-24 CRAN (R 3.4.3)                    
#>  htmltools    0.3.6      2017-04-28 CRAN (R 3.4.0)                    
#>  httr         1.3.1      2017-08-20 CRAN (R 3.4.1)                    
#>  jsonlite     1.5        2017-06-01 CRAN (R 3.4.0)                    
#>  knitr        1.20       2018-02-20 CRAN (R 3.4.3)                    
#>  lattice      0.20-35    2017-03-25 CRAN (R 3.4.2)                    
#>  lazyeval     0.2.1      2017-10-29 CRAN (R 3.4.2)                    
#>  lubridate    1.7.2      2018-02-06 CRAN (R 3.4.3)                    
#>  magrittr     1.5        2014-11-22 CRAN (R 3.4.2)                    
#>  memoise      1.1.0      2018-01-08 Github (hadley/memoise@611cfad)   
#>  methods    * 3.4.3      2017-12-06 local                             
#>  mnormt       1.5-5      2016-10-15 CRAN (R 3.4.1)                    
#>  modelr       0.1.1      2017-07-24 CRAN (R 3.4.1)                    
#>  munsell      0.4.3      2016-02-13 CRAN (R 3.4.2)                    
#>  nlme         3.1-131.1  2018-02-16 CRAN (R 3.4.3)                    
#>  opencage   * 0.1.4.9001 2018-02-26 local                             
#>  pacman     * 0.4.6      2017-05-14 CRAN (R 3.4.0)                    
#>  parallel     3.4.3      2017-12-06 local                             
#>  pillar       1.2.0      2018-02-26 CRAN (R 3.4.3)                    
#>  pkgconfig    2.0.1      2017-03-21 CRAN (R 3.4.0)                    
#>  plyr         1.8.4      2016-06-08 CRAN (R 3.4.2)                    
#>  psych        1.7.8      2017-09-09 CRAN (R 3.4.1)                    
#>  purrr      * 0.2.4      2017-10-18 CRAN (R 3.4.2)                    
#>  R6           2.2.2      2017-06-17 CRAN (R 3.4.0)                    
#>  Rcpp         0.12.15    2018-01-20 CRAN (R 3.4.3)                    
#>  readr      * 1.2.0      2018-01-17 Github (tidyverse/readr@377c9ce)  
#>  readxl       1.0.0      2017-04-18 CRAN (R 3.4.2)                    
#>  reshape2     1.4.3      2017-12-11 CRAN (R 3.4.2)                    
#>  rlang        0.2.0      2018-02-20 CRAN (R 3.4.3)                    
#>  rmarkdown    1.8        2017-11-17 CRAN (R 3.4.2)                    
#>  rprojroot    1.3-2      2018-01-03 CRAN (R 3.4.3)                    
#>  rstudioapi   0.7        2017-09-07 CRAN (R 3.4.1)                    
#>  rvest        0.3.2      2016-06-17 CRAN (R 3.4.2)                    
#>  scales       0.5.0.9000 2017-09-07 Github (hadley/scales@d767915)    
#>  stats      * 3.4.3      2017-12-06 local                             
#>  stringi      1.1.6      2017-11-17 CRAN (R 3.4.2)                    
#>  stringr    * 1.3.0      2018-02-19 CRAN (R 3.4.3)                    
#>  tibble     * 1.4.2      2018-01-22 CRAN (R 3.4.3)                    
#>  tidyr      * 0.8.0      2018-01-29 CRAN (R 3.4.3)                    
#>  tidyselect   0.2.3      2017-11-06 CRAN (R 3.4.2)                    
#>  tidyverse  * 1.2.1      2017-11-14 CRAN (R 3.4.2)                    
#>  tools        3.4.3      2017-12-06 local                             
#>  triebeard    0.3.0      2016-08-04 CRAN (R 3.4.0)                    
#>  urltools     1.7.0      2018-01-20 CRAN (R 3.4.3)                    
#>  utf8         1.1.3      2018-01-03 CRAN (R 3.4.3)                    
#>  utils      * 3.4.3      2017-12-06 local                             
#>  withr        2.1.1      2017-12-19 CRAN (R 3.4.3)                    
#>  xml2         1.2.0      2018-01-24 CRAN (R 3.4.3)                    
#>  yaml         2.1.16     2017-12-12 CRAN (R 3.4.3)
```

</details>
