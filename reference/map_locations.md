# Maps of media records

`map_locations` creates maps to visualize the geographic spread of media
records.

## Usage

``` r
map_locations(
  metadata,
  cluster = FALSE,
  marker_color = NULL,
  by = "species",
  type = c("circles", "markers"),
  palette = function(n) grDevices::hcl.colors(n, "mako"),
  tags = c("repository", "key", "species", "date", "country", "locality", "user_name"),
  show_media = TRUE,
  popup_size = 1
)
```

## Arguments

- metadata:

  Data frame with the metadata of the media records to be mapped.
  Typically the output of one of the query functions in this package
  (e.g.
  [`query_gbif()`](https://docs.ropensci.org/suwo/reference/query_gbif.md),
  [`query_inaturalist()`](https://docs.ropensci.org/suwo/reference/query_inaturalist.md),
  etc.) or metadata formatting functions (e.g.
  [`merge_metadata()`](https://docs.ropensci.org/suwo/reference/merge_metadata.md),
  [`remove_duplicates()`](https://docs.ropensci.org/suwo/reference/remove_duplicates.md),
  etc.). Note that only observations with geographic coordinates (i.e.,
  non-missing values in the `latitude` and `longitude` columns) are
  displayed in the map.

- cluster:

  Logical to control if icons are clustered by locality. Default is
  `FALSE`. Only applies when `type = "markers"`. When `cluster = TRUE`,
  markers that are close together will be clustered into a single marker
  that shows the number of observations in that cluster. Users can click
  on the cluster marker to zoom in and see the individual markers.

- marker_color:

  Character vector with the color(s) to be used for the markers (when
  `type = "markers"`. Possible values are "red", "darkred", "lightred",
  "orange", "beige", "green", "darkgreen", "lightgreen", "blue",
  "darkblue","lightblue", "purple", "darkpurple", "pink", "cadetblue",
  "white", "gray", "lightgray", "black". Can be used to indicate the
  levels of a character or factor column with the argument "by". In such
  a case users must supplied as many colors as levels in the column.

- by:

  Character string indicating the name of the column used to group
  observations for coloring. Default is `"species"`. For
  `type = "circles"`, this determines the color mapping and legend. For
  `type = "markers"`, this determines how marker colors are assigned.

- type:

  Character string indicating how observations are displayed. Options
  are `"circles"` (default) or `"markers"`. Circles use a color palette
  and include a legend, while markers use colored icons and can be
  clustered.

- palette:

  Function used to generate colors for circle markers when
  `type = "circles"`. The function must take a single integer (`n`) and
  return `n` colors. By default it uses
  `function(n) grDevices::hcl.colors(n, "mako")`. Ignored when
  `type = "markers"`.

- tags:

  Character vector with the names of the columns in `metadata` to be
  shown in the popup. Default is
  `c("repository", "key", "species", "date", "country", "locality", "user_name")`.

- show_media:

  Logical indicating whether to display media files (audio, images,
  videos) in the popup windows. Default is `TRUE`.

- popup_size:

  Numeric value that controls the size of the popups. Default is `1`.
  Values greater than `1` will increase the size of the popups, while
  values between `0` and `1` will decrease it.

## Value

An a `leaflet` interactive map object with the locations of the
observations.

## Details

The function uses the `leaflet` package to create interactive maps for
visualizing the geographic spread of observations. Note that only
observations with geographic coordinates are displayed. For each
observation the function displays a marker in the map with a popup that
shows the species name, country, locality, user name, and repository.
The popup also includes an audio player for sound recordings, an image
for photos and a video player for videos.

When multiple media files are associated with the same observation
(i.e., identical values in the `key` column), they are grouped into a
single popup. The first media item is displayed by default, and users
can navigate through additional media using arrow buttons within the
popup.

Users can zoom in and out of the map and click on the markers to see the
popups. If `cluster = TRUE`, markers that are close together will be
clustered into a single marker that shows the number of observations in
that cluster. Users can click on the cluster marker to zoom in and see
the individual markers. This function is useful for exploring the
geographic distribution of media records and identifying patterns or
gaps in the data.

Check the [leaflet package
documentation](https://rstudio.github.io/leaflet/index.html) and the
[leaflet.extras
package](https://CRAN.R-project.org/package=leaflet.extras) for more
information on how to customize maps.

## Author

Marcelo Araya-Salas (<marcelo.araya@ucr.ac.cr>)

## Examples

``` r
if(interactive()){
# search in xeno-canto
e_hochs <- query_gbif(species = "Entoloma hochstetteri", format = "image")

# run if query didnt fail
if (!is.null(e_hochs)) {

  # create map
  map_locations(e_hochs)

  # create map without media
  map_locations(e_hochs, show_media = FALSE)
}
}
```
