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

places_df <- data.frame(places, stringsAsFactors = FALSE) %>% rowid_to_column()
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
#> 1          9 Arc de Tri~ 31UDQ~ JN18du59jq          33 "\U0~ u09wh1~   119
#> 2          6 Paris, Fra~ 31UDQ~ JN18eu14ut          33 "\U0~ u09tvm~   119
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
    oc = oc_forward(places, limit = 1, output = "df_list")) %>% 
  tidyr::unnest(oc) %>% class()
#> [1] "data.frame"
```

return dataframe, multiple results, no\_annotations
---------------------------------------------------

``` r

places_df %>% 
  mutate(
    oc = oc_forward(places, output = "df_list", no_annotations = TRUE)) %>% 
  tidyr::unnest(oc)
#>   rowid                             places confidence
#> 1     1           Elbphilharmonie, Hamburg          9
#> 2     1           Elbphilharmonie, Hamburg          3
#> 3     2          Baker Street 221b, London          9
#> 4     2          Baker Street 221b, London         10
#> 5     2          Baker Street 221b, London          1
#> 6     3 Arc de Triomphe de l’Etoile, Paris          9
#> 7     3 Arc de Triomphe de l’Etoile, Paris          6
#>                                                                       formatted
#> 1 Elbe Philharmonic Hall, Platz der Deutschen Einheit 1, 20457 Hamburg, Germany
#> 2                                                              Hamburg, Germany
#> 3     Sherlock Holmes Museum, 221B Baker Street, London NW1 6XE, United Kingdom
#> 4                             221B Baker Street, London NW1 6XE, United Kingdom
#> 5                                        London, Greater London, United Kingdom
#> 6                 Arc de Triomphe, Place Charles de Gaulle, 75008 Paris, France
#> 7                                                                 Paris, France
#>   northeast_lat northeast_lng southwest_lat southwest_lng
#> 1      53.54162     9.9849674      53.54088     9.9832434
#> 2      53.71523    10.2479060      53.41517     9.7347385
#> 3      51.52381    -0.1584445      51.52371    -0.1585445
#> 4      51.52379    -0.1584418      51.52373    -0.1585686
#> 5      51.69138     0.3356759      51.28622    -0.5088138
#> 6      48.87404     2.2953512      48.87352     2.2947196
#> 7      48.90165     2.4163420      48.81586     2.2237277
#>   ISO_3166_1_alpha_2                      type            arts_centre
#> 1                 DE               arts_centre Elbe Philharmonic Hall
#> 2                 DE                      city                   <NA>
#> 3                 GB                    museum                   <NA>
#> 4                 GB                  building                   <NA>
#> 5                 GB                      city                   <NA>
#> 6                 FR                attraction                   <NA>
#> 7                 FR local_administrative_area                   <NA>
#>                 city_district        country country_code house_number
#> 1               Hamburg-Mitte        Germany           de            1
#> 2                        <NA>        Germany           de         <NA>
#> 3                        <NA> United Kingdom           gb         221B
#> 4                        <NA> United Kingdom           gb         221B
#> 5                        <NA> United Kingdom           gb         <NA>
#> 6 8th Arrondissement of Paris         France           fr         <NA>
#> 7                        <NA>         France           fr         <NA>
#>                    pedestrian political_union postcode         state
#> 1 Platz der Deutschen Einheit  European Union    20457       Hamburg
#> 2                        <NA>  European Union     <NA>       Hamburg
#> 3                        <NA>  European Union  NW1 6XE       England
#> 4                        <NA>  European Union  NW1 6XE       England
#> 5                        <NA>  European Union     <NA>       England
#> 6     Place Charles de Gaulle  European Union    75008 Ile-de-France
#> 7                        <NA>  European Union     <NA> Île-de-France
#>       suburb    town      lat        lng   city                 museum
#> 1  HafenCity    <NA> 53.54129  9.9842058   <NA>                   <NA>
#> 2       <NA> Hamburg 53.57532 10.0153400   <NA>                   <NA>
#> 3 Marylebone    <NA> 51.52376 -0.1584945 London Sherlock Holmes Museum
#> 4 Marylebone    <NA> 51.52376 -0.1585052 London                   <NA>
#> 5       <NA>  London 51.50853 -0.1257400   <NA>                   <NA>
#> 6   Chaillot    <NA> 48.87378  2.2950372  Paris                   <NA>
#> 7       <NA>   Paris 48.85341  2.3488000   <NA>                   <NA>
#>           road state_district         county      attraction
#> 1         <NA>           <NA>           <NA>            <NA>
#> 2         <NA>           <NA>           <NA>            <NA>
#> 3 Baker Street Greater London           <NA>            <NA>
#> 4 Baker Street Greater London           <NA>            <NA>
#> 5         <NA>           <NA> Greater London            <NA>
#> 6         <NA>           <NA>          Paris Arc de Triomphe
#> 7         <NA>           <NA>          Paris            <NA>
#>   local_administrative_area
#> 1                      <NA>
#> 2                      <NA>
#> 3                      <NA>
#> 4                      <NA>
#> 5                      <NA>
#> 6                      <NA>
#> 7                     Paris
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
#> 1          9 Elbe Philh~ 32UNE~ JO43xm89cv          49 "\U0~ u1x0e4~   133
#> 2          9 Sherlock H~ 30UXC~ IO91wm05xq          44 "\U0~ gcpvh7~   119
#> 3          9 Arc de Tri~ 31UDQ~ JN18du59jq          33 "\U0~ u09wh1~   119
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
  data.frame(places2, stringsAsFactors = FALSE) %>%
  rowid_to_column()
```

``` r

places2_df %>% 
  mutate(
    oc = oc_forward(places2, output = "df_list", no_annotations = TRUE)) %>% 
  tidyr::unnest(oc)
#>    rowid                            places2 confidence
#> 1      1           Elbphilharmonie, Hamburg          9
#> 2      1           Elbphilharmonie, Hamburg          3
#> 3      2          Baker Street 221b, London          9
#> 4      2          Baker Street 221b, London         10
#> 5      2          Baker Street 221b, London          1
#> 6      3 Arc de Triomphe de l’Etoile, Paris          9
#> 7      3 Arc de Triomphe de l’Etoile, Paris          6
#> 8      4           Elbphilharmonie, Hamburg          9
#> 9      4           Elbphilharmonie, Hamburg          3
#> 10     5                                            NA
#> 11     6                            dpprdan         NA
#>                                                                        formatted
#> 1  Elbe Philharmonic Hall, Platz der Deutschen Einheit 1, 20457 Hamburg, Germany
#> 2                                                               Hamburg, Germany
#> 3      Sherlock Holmes Museum, 221B Baker Street, London NW1 6XE, United Kingdom
#> 4                              221B Baker Street, London NW1 6XE, United Kingdom
#> 5                                         London, Greater London, United Kingdom
#> 6                  Arc de Triomphe, Place Charles de Gaulle, 75008 Paris, France
#> 7                                                                  Paris, France
#> 8  Elbe Philharmonic Hall, Platz der Deutschen Einheit 1, 20457 Hamburg, Germany
#> 9                                                               Hamburg, Germany
#> 10                                                                          <NA>
#> 11                                                                          <NA>
#>    northeast_lat northeast_lng southwest_lat southwest_lng
#> 1       53.54162     9.9849674      53.54088     9.9832434
#> 2       53.71523    10.2479060      53.41517     9.7347385
#> 3       51.52381    -0.1584445      51.52371    -0.1585445
#> 4       51.52379    -0.1584418      51.52373    -0.1585686
#> 5       51.69138     0.3356759      51.28622    -0.5088138
#> 6       48.87404     2.2953512      48.87352     2.2947196
#> 7       48.90165     2.4163420      48.81586     2.2237277
#> 8       53.54162     9.9849674      53.54088     9.9832434
#> 9       53.71523    10.2479060      53.41517     9.7347385
#> 10            NA            NA            NA            NA
#> 11            NA            NA            NA            NA
#>    ISO_3166_1_alpha_2                      type            arts_centre
#> 1                  DE               arts_centre Elbe Philharmonic Hall
#> 2                  DE                      city                   <NA>
#> 3                  GB                    museum                   <NA>
#> 4                  GB                  building                   <NA>
#> 5                  GB                      city                   <NA>
#> 6                  FR                attraction                   <NA>
#> 7                  FR local_administrative_area                   <NA>
#> 8                  DE               arts_centre Elbe Philharmonic Hall
#> 9                  DE                      city                   <NA>
#> 10               <NA>                      <NA>                   <NA>
#> 11               <NA>                      <NA>                   <NA>
#>                  city_district        country country_code house_number
#> 1                Hamburg-Mitte        Germany           de            1
#> 2                         <NA>        Germany           de         <NA>
#> 3                         <NA> United Kingdom           gb         221B
#> 4                         <NA> United Kingdom           gb         221B
#> 5                         <NA> United Kingdom           gb         <NA>
#> 6  8th Arrondissement of Paris         France           fr         <NA>
#> 7                         <NA>         France           fr         <NA>
#> 8                Hamburg-Mitte        Germany           de            1
#> 9                         <NA>        Germany           de         <NA>
#> 10                        <NA>           <NA>         <NA>         <NA>
#> 11                        <NA>           <NA>         <NA>         <NA>
#>                     pedestrian political_union postcode         state
#> 1  Platz der Deutschen Einheit  European Union    20457       Hamburg
#> 2                         <NA>  European Union     <NA>       Hamburg
#> 3                         <NA>  European Union  NW1 6XE       England
#> 4                         <NA>  European Union  NW1 6XE       England
#> 5                         <NA>  European Union     <NA>       England
#> 6      Place Charles de Gaulle  European Union    75008 Ile-de-France
#> 7                         <NA>  European Union     <NA> Île-de-France
#> 8  Platz der Deutschen Einheit  European Union    20457       Hamburg
#> 9                         <NA>  European Union     <NA>       Hamburg
#> 10                        <NA>            <NA>     <NA>          <NA>
#> 11                        <NA>            <NA>     <NA>          <NA>
#>        suburb    town      lat        lng   city                 museum
#> 1   HafenCity    <NA> 53.54129  9.9842058   <NA>                   <NA>
#> 2        <NA> Hamburg 53.57532 10.0153400   <NA>                   <NA>
#> 3  Marylebone    <NA> 51.52376 -0.1584945 London Sherlock Holmes Museum
#> 4  Marylebone    <NA> 51.52376 -0.1585052 London                   <NA>
#> 5        <NA>  London 51.50853 -0.1257400   <NA>                   <NA>
#> 6    Chaillot    <NA> 48.87378  2.2950372  Paris                   <NA>
#> 7        <NA>   Paris 48.85341  2.3488000   <NA>                   <NA>
#> 8   HafenCity    <NA> 53.54129  9.9842058   <NA>                   <NA>
#> 9        <NA> Hamburg 53.57532 10.0153400   <NA>                   <NA>
#> 10       <NA>    <NA>       NA         NA   <NA>                   <NA>
#> 11       <NA>    <NA>       NA         NA   <NA>                   <NA>
#>            road state_district         county      attraction
#> 1          <NA>           <NA>           <NA>            <NA>
#> 2          <NA>           <NA>           <NA>            <NA>
#> 3  Baker Street Greater London           <NA>            <NA>
#> 4  Baker Street Greater London           <NA>            <NA>
#> 5          <NA>           <NA> Greater London            <NA>
#> 6          <NA>           <NA>          Paris Arc de Triomphe
#> 7          <NA>           <NA>          Paris            <NA>
#> 8          <NA>           <NA>           <NA>            <NA>
#> 9          <NA>           <NA>           <NA>            <NA>
#> 10         <NA>           <NA>           <NA>            <NA>
#> 11         <NA>           <NA>           <NA>            <NA>
#>    local_administrative_area
#> 1                       <NA>
#> 2                       <NA>
#> 3                       <NA>
#> 4                       <NA>
#> 5                       <NA>
#> 6                       <NA>
#> 7                      Paris
#> 8                       <NA>
#> 9                       <NA>
#> 10                      <NA>
#> 11                      <NA>
```

Remaining Problems
==================

-   \[ \] countrycode per row not yet possible (need to change vectorisation in `oc_forward()` from `purrr::map` to `purrr::pmap`)
-   \[ \] `NA_character` as placename (returns results for `placename = "NA"`)
-   \[ \] `add_query` parameter (use placename directly from R, not via API roundtrip)
-   \[ \] handle (HTTP) errors while querying: `purrr::safely`?!
