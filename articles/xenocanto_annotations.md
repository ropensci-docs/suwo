# Xeno-Canto annotations

The goal of this vignette is to illustrate how to query for and retrieve
Xeno-Canto annotation data using
[query_xenocanto()](https://docs.ropensci.org/suwo/reference/query_xenocanto.html)

Xeno-Canto recordings are sometimes annotated: one or more sounds in the
sound file are individually marked with their own species
identification, time and frequency bounds, sound type, and other
details.

suwo (version \>= 0.2.3) now allows users to retrieved annotations along
with the associated sound files. Whenever the recordings matching a
search have annotations attached, they are automatically extracted and
made available as an attribute of the result.

## Basic usage

[query_xenocanto()](https://docs.ropensci.org/suwo/reference/query_xenocanto.html)
requires a Xeno-Canto API key. Get yours at
<https://xeno-canto.org/account>, then set it as an environment variable
named `xc_api_key`:

``` r

# run this in the console, don't save it in a script
Sys.setenv(xc_api_key = "YOUR_API_KEY_HERE")
```

A regular call to
[query_xenocanto()](https://docs.ropensci.org/suwo/reference/query_xenocanto.html)
works exactly as usual:

``` r

pippip <- query_xenocanto(species = "Pipistrellus pipistrellus")
```

    ℹ Obtaining metadata: 
    ✔ 1154 matching sound files found 🎊
    ✔ 2 annotations found and added to the returned object's `annotations` attribute (access with attr(<your_object>, "annotations")) 🌈

The main result is the same data frame returned by any of the suwo’s
query function, one row per recording:

``` r

head(pippip[, c("key", "species", "country", "locality")], 4)
```

    ##       key                   species     country
    ## 1 1165950 Pipistrellus pipistrellus      Poland
    ## 2 1164494 Pipistrellus pipistrellus Netherlands
    ## 3 1154947 Pipistrellus pipistrellus Netherlands
    ## 4 1154946 Pipistrellus pipistrellus Netherlands
    ##                                                                         locality
    ## 1                  Gmina Pabianice (near  Pabianice), Powiat pabianicki, Łódzkie
    ## 2             Zoetermeer, Meerzicht (near  Zoetermeer), Zoetermeer, Zuid-Holland
    ## 3 Cadier en Keer, Kloosterpad (near  Cadier en Keer), Eijsden-Margraten, Limburg
    ## 4 Cadier en Keer, Kloosterpad (near  Cadier en Keer), Eijsden-Margraten, Limburg

Whenever any of the matching recordings have annotations attached, a
second message reports how many were found and confirms they were added
to the result as an attribute. If none of the matching recordings have
annotations, this message is simply not shown, and the attribute is
`NULL`.

## Accessing the annotations

Retrieve the annotations with
[`attr()`](https://rdrr.io/r/base/attr.html):

``` r

annotations <- attr(pippip, "annotations")

nrow(annotations)
```

    ## [1] 2

Each row is one annotated segment, not one recording. A single recording
can have several annotations, each describing a different part of that
one sound file:

``` r

annotations[, c("key", "species", "start_time", "end_time", "frequency_low", "frequency_high")]
```

    ##       key                   species start_time end_time frequency_low
    ## 1 1040808 Pipistrellus pipistrellus   0.010345  6.05997         14719
    ## 2 1040808 Pipistrellus pipistrellus   2.580910  4.60631         45469
    ##   frequency_high
    ## 1          48281
    ## 2          66281

## Understanding the annotation columns

The full set of columns returned includes:

``` r

names(annotations)
```

    ##  [1] "annotation_xc_id"             "key"                         
    ##  [3] "species"                      "annotator"                   
    ##  [5] "start_time"                   "end_time"                    
    ##  [7] "frequency_high"               "frequency_low"               
    ##  [9] "sound_type"                   "sex"                         
    ## [11] "life_stage"                   "annotation_remarks"          
    ## [13] "original_set_name"            "original_set_creator"        
    ## [15] "original_set_owner"           "original_set_license"        
    ## [17] "original_set_uri"             "original_set_creation_date"  
    ## [19] "annotation_set_name"          "annotation_set_creator"      
    ## [21] "annotation_set_creation_date" "annotation_set_remarks"      
    ## [23] "file_url"                     "observation_url"

A few worth highlighting:

- **`key`**. The Xeno-Canto ID of the *recording* this annotation
  belongs to. This matches the `key` column of the main result, and is
  the column to join on (see below).
- **`species`**. The species identified in this specific annotated
  segment. This is usually, but not necessarily always, the same as the
  parent recording’s overall species. It can differ for an annotated
  background call of a different species.
- **`start_time`** / **`end_time`**. The annotated segment’s time bounds
  within the recording, in seconds.
- **`frequency_low`** / **`frequency_high`**. The annotated segment’s
  frequency bounds, in Hz.
- **`annotator`**. Who created this specific annotation.
- **`sound_type`**, **`sex`**, **`life_stage`**,
  **`annotation_remarks`**. Further details about the annotated segment,
  when available.
- **`annotation_set_name`**, **`annotation_set_creator`**,
  **`annotation_set_creation_date`**, **`annotation_set_remarks`**.
  Metadata about the *set* of annotations this one belongs to (a
  recording’s annotations are usually created together, as one set).
- **`file_url`** / **`observation_url`**. Links back to the parent
  recording.

Since several annotation rows commonly share the same `key` (several
segments from the same recording), values in `key` and `species` are
expected to repeat. This reflects the data correctly rather than
indicating a problem.

## Searching specifically for annotated recordings

Annotation data is available on any query’s result whenever present,
regardless of the search used. To instead search specifically for
recordings that have an annotation matching certain criteria, use
Xeno-Canto’s `ann_*` search tags directly in the `species` argument,
e.g. `ann_sp` for the annotated species, `ann_type` for the annotated
sound type:

``` r

# recordings with an annotation identified as this species
corcor <- query_xenocanto(species = 'ann_sp:"Corvus corax"')
```

    ℹ Obtaining metadata: 
    ✔ 153 matching sound files found 🎊
    ✔ 491 annotations found and added to the returned object's `annotations` attribute (access with attr(<your_object>, "annotations")) 😸

This only changes *which recordings* are returned by the search, but
does not change how annotation data itself is retrieved or structured.
The `annotations` attribute is extracted the same way either way.

### Landscape recordings

Some annotations come from **soundscape/landscape recordings** rather
than a recording of a single focal species. Xeno-Canto labels the
annotated species in these cases as `Sonus naturalis` (not a real
biological taxon, but a placeholder Xeno-Canto uses specifically for
this recording type). This can show up unexpectedly when inspecting
annotation species, even for a search targeting a real species:

``` r

table(attr(corcor, "annotations")$species)
```

    ## 
    ##         Acanthis cabaret         Acanthis flammea      Acanthis hornemanni 
    ##                        8                        8                        1 
    ##       Accipiter gentilis        Anthus spinoletta         Anthus trivialis 
    ##                        2                        2                        4 
    ##              Buteo buteo      Carduelis carduelis       Certhia familiaris 
    ##                        1                        1                        1 
    ##             Corvus corax      Cyanistes caeruleus        Dendrocopos major 
    ##                      268                        6                       11 
    ##       Erithacus rubecula        Fringilla coelebs Fringilla montifringilla 
    ##                        3                       27                        3 
    ##    Lophophanes cristatus        Loxia curvirostra        Motacilla cinerea 
    ##                        3                        2                        4 
    ##              Parus major           Periparus ater     Phoenicurus ochruros 
    ##                        9                        7                       37 
    ##   Phylloscopus collybita  Phylloscopus sibilatrix   Phylloscopus trochilus 
    ##                        7                        4                        2 
    ##              Picus canus         Poecile montanus        Prunella collaris 
    ##                        3                        7                        6 
    ##       Prunella modularis     Pyrrhocorax graculus      Regulus ignicapilla 
    ##                        8                       14                        1 
    ##          Regulus regulus           Sitta europaea            Spinus spinus 
    ##                        4                        3                        4 
    ##       Sylvia atricapilla        Tetrastes bonasia  Troglodytes troglodytes 
    ##                        2                        1                        3 
    ##        Turdus philomelos         Turdus torquatus        Turdus viscivorus 
    ##                        6                        1                        7

Keep this in mind when filtering or summarizing annotations by species.
`Sonus naturalis` entries describe general soundscape content picked up
in the background, not an actual species identification for that
segment. To search specifically for this kind of recording, use
`grp:soundscape` (or `grp:0`). See [Searching non-bird
taxa](#searching-non-bird-taxa) below.

### Searching non-bird taxa

The `grp:` tag narrows a search to a specific taxonomic group, and is
especially useful combined with other tags. Valid values are
`grp:birds`, `grp:grasshoppers`, `grp:bats`, `grp:frogs`, and
`grp:"land mammals"` (note the quotes, needed because the value contains
a space). Numeric IDs work too (`1` through `5` for the five groups
above, e.g. `grp:2` is equivalent to `grp:grasshoppers`). Soundscape
recordings are a special case, since they can include multiple groups at
once. Use `grp:soundscape` or `grp:0` to search those specifically.

Including `grp:` matters in particular when combined with `ann_*`
annotation tags: omitting it for a non-bird species alongside an
annotation tag can cause the request to fail outright with a server
error, rather than simply returning no results:

``` r

# fails with an HTTP 500 server error (grasshoppers require grp: to be
# specified explicitly when searching by annotation tags)
query_xenocanto(species = 'ann_sp:"Gryllus campestris"')
```

Including `grp:` resolves it:

``` r

grycamp <- query_xenocanto(
  species = 'grp:grasshoppers ann_sp:"Gryllus campestris"'
)
```

    ℹ Obtaining metadata: 
    ✔ 4 matching sound files found 🎊
    ✔ 10 annotations found and added to the returned object's `annotations` attribute (access with attr(<your_object>, "annotations")) 🌈

The `ann:yes` tag searches for recordings that have *any* annotation
attached, regardless of what species or type it identifies. This is
unlike the `ann_*` tags (`ann_sp`, `ann_type`, etc.), which filter by a
specific annotation attribute. Combined with `grp:`, this is a
convenient way to browse annotated recordings across a whole non-bird
group:

``` r

all_bats <- query_xenocanto(species = "ann:yes grp:bats")

all_frogs <- query_xenocanto(species = "ann:yes grp:frogs")
```

Annotations can then be easily formatted to be used by other R packages
like [warbleR](https://CRAN.R-project.org/package=warbleR) or
[Rraven](https://CRAN.R-project.org/package=Rraven) for further
manipulation and analysis.

For more details on Xeno-Canto’s annotation system, see their [official
documentation](https://xeno-canto.org/article/318).

#### Session information

Click to see

    ## R version 4.6.0 (2026-04-24)
    ## Platform: x86_64-pc-linux-gnu
    ## Running under: Ubuntu 24.04.4 LTS
    ## 
    ## Matrix products: default
    ## BLAS:   /usr/lib/x86_64-linux-gnu/openblas-pthread/libblas.so.3 
    ## LAPACK: /usr/lib/x86_64-linux-gnu/openblas-pthread/libopenblasp-r0.3.26.so;  LAPACK version 3.12.0
    ## 
    ## locale:
    ##  [1] LC_CTYPE=en_US.UTF-8       LC_NUMERIC=C              
    ##  [3] LC_TIME=en_US.UTF-8        LC_COLLATE=en_US.UTF-8    
    ##  [5] LC_MONETARY=en_US.UTF-8    LC_MESSAGES=en_US.UTF-8   
    ##  [7] LC_PAPER=en_US.UTF-8       LC_NAME=C                 
    ##  [9] LC_ADDRESS=C               LC_TELEPHONE=C            
    ## [11] LC_MEASUREMENT=en_US.UTF-8 LC_IDENTIFICATION=C       
    ## 
    ## time zone: Etc/UTC
    ## tzcode source: system (glibc)
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ## [1] suwo_0.2.2
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] digest_0.6.39     desc_1.4.3        R6_2.6.1          fastmap_1.2.0    
    ##  [5] xfun_0.60         cachem_1.1.0      knitr_1.52        htmltools_0.5.9  
    ##  [9] rmarkdown_2.32    lifecycle_1.0.5   cli_3.6.6         sass_0.4.10      
    ## [13] pkgdown_2.2.1     textshaping_1.0.5 jquerylib_0.1.4   systemfonts_1.3.2
    ## [17] compiler_4.6.0    tools_4.6.0       ragg_1.5.2        bslib_0.12.0     
    ## [21] evaluate_1.0.5    yaml_2.3.12       otel_0.2.0        jsonlite_2.0.0   
    ## [25] rlang_1.3.0       fs_2.1.0          htmlwidgets_1.6.4
