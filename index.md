# suwo: access nature media repositories

The [suwo](https://docs.ropensci.org/suwo/) package aims to simplify the
retrieval of nature media (mostly photos, audio files and videos) across
multiple online biodiversity databases. The five major media
repositories accessed by this package (GBIF, iNaturalist, Macaulay
Library, WikiAves, and Xeno-Canto) collectively host more than 250
million media files. Such media are increasingly used in diverse fields,
ranging from ecology and evolutionary biology (e.g., trait evolution) to
wildlife monitoring and conservation (e.g., for training species
detection models). The ability to access and download large amounts of
media files and their associated metadata from a single interface thus
provides a uniquely powerful resource for facilitating research and
conservation efforts.

The main features of the package are:

- Obtaining media metadata from online repositories
- Downloading associated media files
- Updating data sets with new records

## Installing suwo

To install the latest developmental version from
[github](https://github.com/) run:

``` r

install.packages("suwo", repos = c(
  'https://ropensci.r-universe.dev',
  'https://cloud.r-project.org'
))

#load package
library(suwo)
```

# Basic workflow for obtaining nature media files

Obtaining nature media using [suwo](https://docs.ropensci.org/suwo/)
follows a basic sequence. The following diagram illustrates this
workflow and the main functions involved:

![Flowchart of the suwo workflow for obtaining nature media files. Step
1, 'Get metadata', includes multiple boxes representing queries to
different repositories, such as query_wikiaves() and query_xenocanto(),
plus additional possible query\_() calls. Arrows from all these queries
converge into Step 2, 'Combine metadata', using merge_metadata() and
'Remove duplicates', using find_duplicates() and remove_duplicates().
The last step is 'Download media', using download_media(). Finally, user
can update previous queries using
update_metadata()](./articles/workflow_diagram.png)

Take a look at the [package
vignette](https://docs.ropensci.org/suwo/articles/suwo.html) for an
overview of the workflow and the core querying functions.

## Intended use and responsible practices

The [suwo](https://docs.ropensci.org/suwo/) package is designed
exclusively for non-commercial, scientific purposes, including research,
education, and conservation. **Commercial use of data or media retrieved
through this package is the user’s responsibility and is allowed only
when the applicable license of the source database explicitly permits
such use, or when explicit, separate permission has been obtained
directly from the original source platforms or rights holders**. Users
must comply with the specific terms of service and data-use policies of
each source database, which may require attribution and may further
restrict commercial application. The package developers assume no
liability for misuse of the retrieved data or for violations of
third-party terms of service.

## Citation

Please cite [suwo](https://docs.ropensci.org/suwo/) as follows:

Araya-Salas M, Elizondo-Calvo J, Rico-Guevara A (2026). *suwo: Access
Nature Media Repositories*. R package version 0.2.2,
<https://docs.ropensci.org/suwo/>.
