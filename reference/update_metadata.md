# Update metadata

`update_metadata` update metadata from previous queries.

## Usage

``` r
update_metadata(
  metadata,
  path = ".",
  cores = getOption("suwo_cores", 1),
  pb = getOption("suwo_pb", TRUE),
  verbose = getOption("suwo_verbose", TRUE),
  api_key = NULL,
  dates = NULL
)
```

## Arguments

- metadata:

  Data frame with the metadata of media records. Typically the output of
  one of the query functions in this package (e.g.
  [`query_gbif()`](https://docs.ropensci.org/suwo/reference/query_gbif.md),
  [`query_inaturalist()`](https://docs.ropensci.org/suwo/reference/query_inaturalist.md),
  etc.).

- path:

  Directory path where the .csv file will be saved. Only applicable for
  [`query_macaulay()`](https://docs.ropensci.org/suwo/reference/query_macaulay.md)
  query results. By default it is saved into the current working
  directory (`"."`).

- cores:

  Numeric vector of length 1. Controls whether parallel computing is
  applied by specifying the number of cores to be used. Default is 1
  (i.e. no parallel computing). Can be set globally for the current R
  session via the "mc.cores" option (e.g. `options(mc.cores = 2)`). Note
  that some repositories might not support parallel queries from the
  same IP address as it might be identified as denial-of-service
  cyberattack.

- pb:

  Logical argument to control if progress bar is shown. Default is
  `TRUE`. Can be set globally for the current R session via the
  "suwo_pb" option ( `options(suwo_pb = TRUE)`). Not shown if only a few
  observations are found.

- verbose:

  Logical argument that determines if text is shown in console. Default
  is `TRUE`. Can be set globally for the current R session via the
  "suwo_verbose" option ( `options(suwo_verbose = TRUE)`).

- api_key:

  Character string referring to the key assigned by Xeno-Canto as
  authorization for searches. Get yours at
  <https://xeno-canto.org/account>. Only needed if the input metadata
  comes from
  [`query_xenocanto()`](https://docs.ropensci.org/suwo/reference/query_xenocanto.md).

- dates:

  Optional numeric vector with years to split the search. If provided,
  the function will perform separate queries for each date range
  (between consecutive date values) and combine the results. Useful for
  queries that return large number of results (i.e. \> 10000 results
  limit). For example, to search for the species between 2010 to 2020
  and between 2021 to 2025 use `dates = c(2010, 2020, 2025)`. If years
  contain decimals searches will be split by months within years as
  well. Only needed if the input metadata comes from
  [`query_macaulay()`](https://docs.ropensci.org/suwo/reference/query_macaulay.md).

## Value

returns a data frame similar to the input 'metadata' with new data
appended.

## Details

This function updates the metadata from a previous query to add entries
found in the source repository. **All observations must belong to the
same repository** (but see examples for code to update metadata from
multiple repositories). The function adds the column `new_entry` which
labels those entries that are new (i.e., not present in the input
metadata). The input data frame must have been obtained from any of the
query functions with the argument `raw_data = FALSE`. The function uses
the same query species and format as in the original query. If no new
entries are found, the function returns the original metadata and prints
a message. If some old entries are not returned in the new query they
are still retained. The function assumes that no new files are added to
existing repository entries. The value of `all_data` (an argument common
to all query functions) is inferred from the columns present in
metadata. If columns beyond the standard output are detected, the
function assumes `all_data = TRUE`. Columns added during processing by
any `suwo` function ("source", "new_entry", "downloaded_file_name",
"download_status", "file_size", "duplicate_group") are ignored to
prevent incorrect inference.

## Author

Marcelo Araya-Salas (<marcelo.araya@ucr.ac.cr>)

## Examples

``` r
# query metadata
a_gioiosa <- query_gbif(species = "Amanita gioiosa", format =  "image")
#> ✔ Obtaining metadata (19 matching records found) 🎊

# run if query didnt fail
if (!is.null(a_gioiosa)) {

# remove the key with more observations
sub_a_gioiosa <-
a_gioiosa[a_gioiosa$key != names(which.max(table(a_gioiosa$key))), ]

# update
up_a_gioiosa <- update_metadata(metadata = sub_a_gioiosa)

# check number of rows is the same (e.g. it has been updated)
nrow(up_a_gioiosa) == nrow(a_gioiosa)

# example multi repository update
# \donttest{
a_orientigemmata <- query_inaturalist(species = "Amanita orientigemmata",
format =  "image")

#remove the key with more observations
sub_a_orientigemmata <-
a_orientigemmata[a_orientigemmata$key !=
names(which.max(table(a_orientigemmata$key))), ]

# merge both metadata
sub_amanitas <- merge_metadata(sub_a_gioiosa, sub_a_orientigemmata)

# split by repository and update separately
up_amanitas_list <-
lapply(split(sub_amanitas, sub_amanitas$repository), update_metadata)

# merge updated metadata
up_amanitas <- do.call(merge_metadata, up_amanitas_list)

 # check number of rows is the same (e.g. it has been updated)
 nrow(up_amanitas) == nrow(a_gioiosa) + nrow(a_orientigemmata)
# }
}
#> ✔ Obtaining metadata (19 matching records found) 🎊
#> ✔ 13 new entries found 🌈
#> ✔ Obtaining metadata (197 matching records found) 🎉
#> ✔ Obtaining metadata (19 matching records found) 😀
#> ✔ 13 new entries found 🥇
#> ✔ Obtaining metadata (197 matching records found) 🥇
#> ✔ 8 new entries found 🎊
#> [1] TRUE
```
