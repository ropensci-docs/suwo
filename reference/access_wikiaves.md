# Obtain WikiAves authentication cookies via a running Chrome or Chromium instance

`access_wikiaves` authenticates with
[WikiAves](https://www.wikiaves.com.br), which sits behind Cloudflare's
bot protection and blocks plain `httr2` requests with an HTTP 403 and a
"Just a moment..." challenge page. This function works around that by
driving a real, visible Chromium-based browser (Google Chrome, Chromium,
or another compatible build) through the Chrome DevTools Protocol (CDP):
it launches the browser (or reuses one already running) pointed at
WikiAves, waits for Cloudflare's JavaScript challenge to clear, and then
extracts the resulting cookies and the user agent string. The values are
packed into a single character string.

By default (`set_env = TRUE`) this string is also stored directly in the
`wikiaves_cookies` environment variable for the current R session, which
is what
[`query_wikiaves()`](https://docs.ropensci.org/suwo/reference/query_wikiaves.md)
reads from by default – so in the common case, calling
`access_wikiaves()` once is enough to authenticate every subsequent
[`query_wikiaves()`](https://docs.ropensci.org/suwo/reference/query_wikiaves.md)
call in the session without passing `cookies` explicitly. See Details.

This function works on Linux, macOS, and Windows. On all three platforms
it will try to auto-detect a suitable browser and a writable temporary
directory if `chrome_bin` and `user_data_dir` are left at their default
(`NULL`); see Details.

## Usage

``` r
access_wikiaves(
  chrome_bin = NULL,
  port = 9333,
  url = "https://www.wikiaves.com.br/",
  timeout = 10,
  launch_wait = 15,
  challenge_wait = 30,
  user_data_dir = NULL,
  set_env = TRUE
)
```

## Arguments

- chrome_bin:

  Character. Name or path of the browser executable to launch. Default
  `NULL`, which triggers auto-detection of a Chrome- or Chromium-based
  browser for the current operating system (see Details). Set this
  explicitly to override auto-detection or to point at a non-standard
  install location, e.g. `access_wikiaves("chromium")` on Linux,
  `"/Applications/Chromium.app/Contents/MacOS/Chromium"` on macOS, or
  `"C:/Program Files/Google/Chrome/Application/chrome.exe"` on Windows.
  Messages printed while the function runs refer to whichever browser
  `chrome_bin` resolves to, so it is easy to tell which one is in use.

- port:

  Integer. The TCP port used for the browser's DevTools remote debugging
  protocol. Default `9333`. If a browser instance is already listening
  on this port, it is reused instead of launching a new one.

- url:

  Character. The WikiAves URL to navigate to and extract cookies from.
  Default `"https://www.wikiaves.com.br/"`. Changing this is only useful
  for testing against a different page on the same domain.

- timeout:

  Numeric. Maximum time, in seconds, to wait for a single round-trip
  response (cookies, user agent, and page title) over the DevTools
  websocket connection. Default `10`.

- launch_wait:

  Numeric. Maximum time, in seconds, to wait for a newly launched
  browser process to open its DevTools debugging port. Default `15`.
  Only relevant when the browser is not already running on `port`.

- challenge_wait:

  Numeric. Maximum total time, in seconds, to keep polling the page
  while Cloudflare's "Just a moment..." challenge is showing. Default
  `30`. If the challenge has not cleared within this window the function
  stops with an informative error; the person can solve the challenge
  manually in the visible browser window (e.g. clicking a checkbox)
  while this function is polling, and it will pick up the cleared page
  on the next check.

- user_data_dir:

  Character. Filesystem path to the browser's user data directory used
  for this session. Default `NULL`, which resolves to a
  `"chrome-wikiaves-profile"` folder inside
  [`tempdir()`](https://rdrr.io/r/base/tempfile.html) – a location valid
  on Linux, macOS, and Windows alike. Reusing the same directory across
  calls lets the browser and Cloudflare recognize the profile as
  previously trusted, which often lets later calls clear the challenge
  automatically without manual interaction.

- set_env:

  Logical. If `TRUE` (the default), also stores the returned cookies
  string in the `wikiaves_cookies` environment variable for the current
  R session via
  [`Sys.setenv()`](https://rdrr.io/r/base/Sys.setenv.html), so that
  [`query_wikiaves()`](https://docs.ropensci.org/suwo/reference/query_wikiaves.md)'s
  default `cookies` argument picks it up automatically without needing
  to pass it explicitly. Set to `FALSE` to make the function a pure
  getter that only returns the cookies string without modifying the
  session's environment variables – useful if you want to manage the
  credentials yourself (e.g. pass them directly into a single
  [`query_wikiaves()`](https://docs.ropensci.org/suwo/reference/query_wikiaves.md)
  call, store them somewhere else, or use a different session's
  variable).

## Value

A single JSON-encoded character string containing `cf_clearance`,
`PHPSESSID`, and `browser_ua`. This is the format expected by the
`cookies` argument of
[`query_wikiaves()`](https://docs.ropensci.org/suwo/reference/query_wikiaves.md).
The string is returned invisibly if `set_env = TRUE` (its side effect –
setting `wikiaves_cookies` – is normally what matters in that case), and
visibly if `set_env = FALSE`. Not expected to be modified by the user.
Returns `invisible(NULL)` instead if no internet connection is
available.

## Details

This function requires the **websocket** and **later** packages, which
are listed under Suggests rather than Imports since this workflow is
only needed for the WikiAves data source and depends on a local Chrome-
or Chromium-based browser installation and a usable display (e.g. the
`DISPLAY` environment variable on Linux). If either package is missing,
in an interactive session the function asks for permission before
installing them via
[`install.packages()`](https://rdrr.io/r/utils/install.packages.html);
declining stops the function with instructions to install manually. In a
non-interactive session (e.g. within `R CMD check`, CI, or `Rscript`),
the function cannot prompt and instead stops immediately with the same
instructions.

**Browser auto-detection.** When `chrome_bin = NULL` (the default), the
function first checks whether the **chromote** package happens to be
installed and, if so, uses its exported
[`chromote::find_chrome()`](https://rstudio.github.io/chromote/reference/find_chrome.html)
to locate a browser – this respects the `CHROMOTE_CHROME` environment
variable and **chromote**'s own per-OS search logic. **chromote** is not
a dependency of this function; if it is not installed, or its detection
does not find anything, the function falls back to its own
candidate-path search:

- **Linux**: `google-chrome`, `google-chrome-stable`, `chromium`, and
  `chromium-browser`, in that order, checked via
  [`Sys.which()`](https://rdrr.io/r/base/Sys.which.html).

- **macOS**:
  `/Applications/Google Chrome.app/Contents/MacOS/Google Chrome` and
  `/Applications/Chromium.app/Contents/MacOS/Chromium`.

- **Windows**: the `chrome.exe` install locations under `Program Files`
  and `Program Files (x86)`, as well as a per-user AppData install
  location.

If none of these are found, the function stops with an error explaining
how to set `chrome_bin` manually. Auto-detected paths have only been
tested against typical default install locations; if your browser is
installed somewhere non-standard, pass its path explicitly via
`chrome_bin`.

On first use (or once the session has expired, which typically happens
within a couple of hours), Cloudflare may present a visible interactive
challenge in the launched browser window. Switching to that window and
completing the challenge is a normal part of the workflow; this function
polls for up to `challenge_wait` seconds so the person has time to do so
before it gives up, and prints a message once the cookies have been
successfully retrieved.

**Automatic environment variable.** By default (`set_env = TRUE`), the
returned cookies string is also written to
`Sys.setenv(wikiaves_cookies = ...)` before the function returns. Since
[`query_wikiaves()`](https://docs.ropensci.org/suwo/reference/query_wikiaves.md)'s
`cookies` argument defaults to `Sys.getenv("wikiaves_cookies")`, this
means a bare `access_wikiaves()` call is normally all that is needed to
authenticate every
[`query_wikiaves()`](https://docs.ropensci.org/suwo/reference/query_wikiaves.md)
call made afterwards in the same R session – there is no need to
manually capture and pass the return value unless `set_env = FALSE` is
used.

## See also

[`query_wikiaves()`](https://docs.ropensci.org/suwo/reference/query_wikiaves.md),
which accepts the output of this function via its `cookies` argument.

## Examples

``` r
if (interactive()) {
# Simplest usage: launches (or reuses) a browser instance,
# auto-detecting an appropriate Chrome/Chromium binary for the current
# OS, solves the Cloudflare challenge if needed, and stores the
# resulting cookies in the `wikiaves_cookies` environment variable
# (set_env = TRUE by default) -- so query_wikiaves() picks them up
# automatically without passing `cookies` explicitly:
access_wikiaves()
result <- query_wikiaves(species = "Procnias averano", format = "sound")

# chrome_bin is the first argument, so a non-default browser binary can
# be supplied positionally:
access_wikiaves("chromium")

# Using a non-default debugging port as well:
access_wikiaves("chromium", port = 9444)

# set_env = FALSE turns access_wikiaves() into a pure getter: nothing
# is stored automatically, and the cookies string must be captured and
# passed to query_wikiaves() (or elsewhere) manually:
cookies_live <- access_wikiaves(set_env = FALSE)
result <- query_wikiaves(
  species = "Procnias averano",
  format = "sound",
  cookies = cookies_live
)
}
```
