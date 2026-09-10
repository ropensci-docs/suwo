# Explore geographic variation

The goal of this vignette is to illustrate how to use the
[map_locations()](https://docs.ropensci.org/suwo/reference/map_locations.html)
function to explore the geographic distribution of media records as a
proxy to understand geographic variation in biological traits.

Understanding the geographic distribution of biological variation is
essential not only for ecological and evolutionary research ([Zamudio et
al. 2016](#ref-zamudio2016)) but also for building robust automated
classification models that account for phenotypic diversity ([Pagano et
al. 2023](#ref-pagano2023)). The
[map_locations()](https://docs.ropensci.org/suwo/reference/map_locations.html)
function in **suwo** allows researchers to interactively explore the
spatial distribution of media records retrieved through the package,
enabling rapid visual assessment of geographic patterns in diverse trait
data.

The function accepts a metadata data frame —typically obtained from any
of the `query_()` functions or as the output of
[merge_metadata()](https://docs.ropensci.org/suwo/reference/merge_metadata.html)
and
[remove_duplicates()](https://docs.ropensci.org/suwo/reference/remove_duplicates.html)
— and generates an interactive map powered by the [leaflet
package](https://CRAN.R-project.org/package=leaflet). Each point on the
map represents a unique observation with valid geographic coordinates.
Clicking on a point opens a popup window displaying the associated media
file(s) and selected metadata fields.

In this vignette, we demonstrate how to leverage the key features of
[map_locations()](https://docs.ropensci.org/suwo/reference/map_locations.html)
through three case studies: mapping image records of a color‑polymorphic
frog, mapping sound recordings of the same species, and mapping video
recordings of a bird’s courtship display.

## Mapping images

The [Strawberry Poison Dart Frog (*Oophaga
pumilio*)](https://en.wikipedia.org/wiki/Strawberry_poison_dart_frog)
exhibits remarkable geographic variation in coloration across its range
in Central America ([Wang and Shaffer 2008](#ref-wang2008)). Populations
display striking differences in dorsal and ventral coloration, ranging
from bright red with blue legs to green, yellow, or even white morphs.
This system provides an excellent opportunity to demonstrate how
[map_locations()](https://docs.ropensci.org/suwo/reference/map_locations.html)
can be used to explore spatial patterns of phenotypic variation using
image records.

First, we query for image records of this species from iNaturalist:

``` r

# get metadata
op_image <- query_inaturalist(
  species = "Oophaga pumilio",
  format = "image"
)
```

The raw query returns all available image records, including some that
fall outside the species’ natural range (e.g., observations from zoos,
captive specimens, or misidentified individuals from other regions). To
focus our analysis on wild populations within their native distribution,
we restrict the dataset to records from Central America. Additionally,
to keep the map manageable for this vignette and avoid overcrowding, we
sample observations evenly across the geographic range by splitting the
latitudinal range in 50 bins and selecting one record per bin. Note that
because some observations have multiple associated images, we sample at
the level of observation keys:

``` r

# restrict only to those in its natural range (Central America)
op_image_ca <- op_image[
  op_image$latitude >= 7 &
    op_image$latitude <= 15 &
    op_image$longitude >= -90 &
    op_image$longitude <= -80 &
    !is.na(op_image$latitude),
]

# sample homogeneously across the geographic range
# split latitude in 50 bins
op_image_ca$latitude_bins <- cut(
  op_image_ca$latitude,
  breaks = seq(7, 15, length.out = 50)
)

# sampled keys
sampled_keys <- op_image_ca$key[!duplicated(op_image_ca$latitude_bins)]
set.seed(123)
op_image_ca_sample <- op_image_ca[op_image_ca$key %in% sampled_keys, ]

# make map
map_locations(
  metadata = op_image_ca_sample
)
```

When users click on any observation circle, a popup window appears
displaying the image(s) associated with that record along with its most
relevant metadata. This functionality allows researchers to visually
inspect the color morph present at each locality, enabling rapid
assessment of geographic patterns in coloration. For observations that
contain multiple images (e.g., different angles or life stages), users
can navigate between them using the arrow buttons at the bottom of the
popup. By default the metadata includes the observation key, which
contains a link to the original record on the repository, which can be
used to explore additional information about the record. Users can
download the image by simply clicking on them.

### Customizing popup appearance

The `popup_size` argument controls the scaling of the popup size. For
example, setting `popup_size = 0.6` displays popups at 60% of its
original size, which can be useful when working with very large images
or when wanting to see more context around each point on the map:

``` r

map_locations(
  metadata = op_image_ca_sample,
  popup_size = 0.6
)
```

### Controlling displayed metadata

The `tags` argument allows users to control which metadata fields are
displayed in the popup. By default,
[map_locations()](https://docs.ropensci.org/suwo/reference/map_locations.html)
shows a set of commonly useful fields (e.g., repository, observation
key, species, date, country, locality, and user name). However, this
behavior can be customized to include only the variables relevant to a
given research question:

``` r

map_locations(
  metadata = op_image_ca_sample,
  tags = c( "key", "date"),
  popup_size = 0.6
)
```

… or even suppress all metadata by setting `tags = NULL`:

``` r

map_locations(
  metadata = op_image_ca_sample,
  tags = NULL,
  popup_size = 0.6
)
```

The map also includes a toggle button (located in the top-right corner)
that can open all popups simultaneously, which is particularly useful
for quickly scanning multiple records:

## Mapping sound recordings

Beyond visual traits, vocalizations often show geographic variation that
can reveal patterns of cultural evolution, cryptic diversity, and
population structure. The [Strawberry Poison Dart
Frog](https://en.wikipedia.org/wiki/Strawberry_poison_dart_frog) is a
great example: its advertisement calls seem to be much less variables
across its range, providing an interesting contrast with its striking
geographic variation in coloration. With
[map_locations()](https://docs.ropensci.org/suwo/reference/map_locations.html),
researchers can map where recordings come from and directly compare
calls across different localities:

``` r

op_sound <- query_inaturalist(
  species = "Oophaga pumilio",
  format = "sound"
)

# make map
map_locations(
  op_sound,
  tags = NULL,
  popup_size = 0.6
)
```

When clicking on a point representing a sound recording, the popup
displays an embedded audio player that allows users to listen to the
recording directly within the map interface. This feature is
particularly valuable for bioacoustic research, as it enables rapid
comparison of vocalizations across geographic space without downloading
files or switching between applications. The audio player supports
standard playback controls including play, pause, and volume adjustment
and allows users to directly download the audio file (although this
might depend on the browser).

## Mapping video recordings

Video recordings capture dynamic behaviors that cannot be fully
appreciated through static images or audio alone. Courtship displays,
locomotion, feeding behavior, and other complex activities are often
best documented through video. To illustrate this functionality, we
examine video recordings of the [Red-capped Manakin (*Ceratopipra
mentalis*)](https://en.wikipedia.org/wiki/Red-capped_manakin), a species
known for its elaborate courtship displays involving rapid
wing-snapping, moonwalking, and coordinated jumps:

``` r

cm_video <- query_macaulay(
  species = "Ceratopipra mentalis",
  format = "video",
  all_data = TRUE
)

# Filter for records containing display behavior
cm_video_display <- cm_video[
  grepl("display", cm_video$behavior, ignore.case = TRUE),
]

# make map
map_locations(
  cm_video_display,
  tags = NULL,
  popup_size = 0.6
)
```

Video popups include an embedded video player that supports playback
directly within the map and allows users to directly download the audio
file (although this might depend on the browser). Note that video files
are typically larger than images or audio files, so loading times may
vary depending on internet connection speed and file size.

## Advanced customization options

The
[map_locations()](https://docs.ropensci.org/suwo/reference/map_locations.html)
function offers several additional arguments for customizing map
appearance and behavior.

### Location markers

By default, observations are displayed as semi-transparent circles,
which work well for visualizing density and avoiding visual clutter.
However, users can instead display observations as markers using the
`type = "markers"` argument:

``` r

# make map
map_locations(
  op_image_ca_sample,
  type = "markers",
  marker_color = "darkblue",
)
```

### Marker clustering

When working with large datasets or when multiple observations occur in
close proximity, markers can overlap and become difficult to
distinguish. The `cluster = TRUE` argument enables marker clustering,
which groups nearby markers into a single cluster marker that displays
the number of observations it contains. Clicking on a cluster zooms in
to reveal the individual markers, providing a cleaner map interface
while preserving access to all data points:

``` r

# make map
map_locations(
  op_image_ca_sample,
  type = "markers",
  marker_color = "darkblue",
  cluster = TRUE
)
```

### Color customization

The `marker_color` argument allows users to change the color of markers
when `type = "markers"`. For circle-based maps (`type = "circles"`),
colors are determined by the `palette` argument, which accepts any color
palette.

### Coloring by categorical variables

The `by` argument allows users to color observations according to a
categorical variable in the metadata. In this example we search for
sound recordings of [howler monkey species (genus
*Alouatta*)](https://en.wikipedia.org/wiki/Howler_monkey) from Xenocanto
and use `by = "species"` to color points according to species identity:

``` r

# search for species of the genus manacus
alouatta_sound <- query_xenocanto("gen:Alouatta")

# make map
map_locations(
  alouatta_sound,
  by = "species",
  palette = grDevices::hcl.colors # viridis palette
)
```

### Suppressing media display

For situations where only metadata information is needed, users can
suppress media display entirely using `show_media = FALSE`. This creates
popups that contain only the metadata specified in the `tags` argument,
without any audio players, images, or video players, resulting in faster
rendering and reduced bandwidth usage.

``` r

# make map
map_locations(
  alouatta_sound,
  by = "species", 
  show_media = FALSE,
  palette = grDevices::hcl.colors
)
```

### Further customization of leaflet maps

The function returns a `leaflet` map object, which can be further
customized using any of the options available in the
[leaflet](https://CRAN.R-project.org/package=leaflet) and
[leaflet.extras](https://CRAN.R-project.org/package=leaflet.extras)
packages. Below is an example of how to add satellite imagery and
OpenStreetMap tiles as basemap options, allowing users to toggle between
them:

``` r

op_map <- map_locations(
  metadata = alouatta_sound,
  tags = c("key", "species"),
  popup_size = 0.6,
  palette = grDevices::hcl.colors # viridis palette  
)

op_map |>
  leaflet::addProviderTiles(
    leaflet::providers$Esri.WorldImagery,
    group = "Satellite"
  ) |>
  leaflet::addProviderTiles(leaflet::providers$OpenStreetMap, group = "Map") |>
  leaflet::addLayersControl(
    baseGroups = c("Satellite", "Map"),
    options = leaflet::layersControlOptions(collapsed = FALSE),
    position = "topleft"
  )
```

Check the [leaflet package
documentation](https://rstudio.github.io/leaflet/index.html) and the
[leaflet.extras
package](https://CRAN.R-project.org/package=leaflet.extras) for more
information on how to customize maps.

#### References

Pagano, Tiago P., Rafael B. Loureiro, Fernanda V. N. Lisboa, et al.
2023. “Bias and Unfairness in Machine Learning Models: A Systematic
Review.” *Big Data and Cognitive Computing* 7 (1): 15.
<https://doi.org/10.3390/bdcc7010015>.

Wang, Ian J., and H. Bradley Shaffer. 2008. “Rapid Color Evolution in an
Aposematic Species: A Phylogenetic Analysis of Color Variation in the
Strikingly Polymorphic Strawberry Poison-Dart Frog.” *Evolution* 62
(11): 2742–59. <https://doi.org/10.1111/j.1558-5646.2008.00507.x>.

Zamudio, Kelly R., R. C. Bell, and N. A. Mason. 2016. “Phenotypes in
Phylogeography.” *PNAS* 113 (29): 8041–48.
<https://doi.org/10.1073/pnas.1602237113>.

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
    ##  [1] cli_3.6.6               knitr_1.52              rlang_1.3.0            
    ##  [4] xfun_0.60               leaflet_2.2.3           otel_0.2.0             
    ##  [7] textshaping_1.0.5       jsonlite_2.0.0          glue_1.8.1             
    ## [10] backports_1.5.1         htmltools_0.5.9         ragg_1.5.2             
    ## [13] sass_0.4.10             scales_1.4.0            rmarkdown_2.32         
    ## [16] crosstalk_1.2.2         evaluate_1.0.5          jquerylib_0.1.4        
    ## [19] fastmap_1.2.0           yaml_2.3.12             lifecycle_1.0.5        
    ## [22] compiler_4.6.0          RColorBrewer_1.1-3      fs_2.1.0               
    ## [25] htmlwidgets_1.6.4       leaflet.providers_3.0.0 farver_2.1.2           
    ## [28] systemfonts_1.3.2       digest_0.6.39           R6_2.6.1               
    ## [31] magrittr_2.0.5          bslib_0.12.0            checkmate_2.3.4        
    ## [34] tools_4.6.0             pkgdown_2.2.1           cachem_1.1.0           
    ## [37] desc_1.4.3
