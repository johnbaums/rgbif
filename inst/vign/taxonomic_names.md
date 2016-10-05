<!--
%\VignetteEngine{knitr::knitr}
%\VignetteIndexEntry{Taxonomic Names}
%\VignetteEncoding{UTF-8}
-->



Taxonomic Names
===============

You have probably, or will, run into problems with taxonomic names. For example,
you may think you know how a taxonomic name is spelled, but then GBIF will not
agree with you. Or, perhaps GBIF will have multiple versions of the taxon,
spelled in slightly different ways. Or, the version of the name that they think
is the _right one_ does not match what yo think is the right one.

This isn't really anyone's fault. It's a result of there not being one accepted
taxonomic source of truth across the globe. There are many different taxonomic
databases. GBIF makes their own _backbone taxonomy_ that they use as a source
of internal truth for taxonomic names. The accepted names in the backbone taxonomy
match those in the database of occurrences - so do try to figure out what
backbone taxonomy version of the name you want.

Another source of problems stems from the fact that names are constantly changing.
Sometimes epithets change, sometimes generic names, and sometimes higher names
like family or tribe. These changes can take a while to work their way in to
GBIF's data.

The following are some examples of confusing name bits. We'll update these if
GBIF's name's change. The difference between each pair of names is highlighted
in bold.

## Load rgbif


```r
library("rgbif")
```

## Helper function

To reduce code duplication, we'll use a little helper function to make a call
to `name_backbone()` for each input name, then `rbind` them together:


```r
name_rbind <- function(..., rank = "species") {
 df <- data.frame(do.call(rbind, lapply(list(...), name_backbone, rank = rank)))
 df[, c('usageKey', 'scientificName', 'canonicalName', 'rank',
       'status', 'confidence', 'matchType', 'synonym')]
}
```

And another function to get the taxonomic data provider


```r
taxon_provider <- function(x) {
  tt <- name_usage(key = x)$data
  datasets(uuid = tt$constituentKey)$data$title
}
```

We use `taxon_provider()` below to get the taxonomy provider in the bulleted list of details
for each taxon (even though you don't see it called, we use it, but the code isn't shown :)).

## Pinus sylvestris vs. P. silvestris


```r
(c1 <- name_rbind("Pinus sylvestris", "Pinus silvestris"))
#>   usageKey      scientificName    canonicalName    rank   status
#> 1  5285637 Pinus sylvestris L. Pinus sylvestris SPECIES ACCEPTED
#> 2  5285637 Pinus sylvestris L. Pinus sylvestris SPECIES ACCEPTED
#>   confidence matchType synonym
#> 1         99     EXACT   FALSE
#> 2         88     FUZZY   FALSE
```

* P. s<b>y</b>lvestris w/ 242570 occurrences - from Catalogue of Life
* P. s<b>i</b>lvestris w/ 242570 occurrences - from Catalogue of Life

## Macrozamia platyrachis vs. M. platyrhachis


```r
(c2 <- name_rbind("Macrozamia platyrachis", "Macrozamia platyrhachis"))
#>   usageKey                      scientificName           canonicalName
#> 1  4928834 Macrozamia platyrachis F. M. Bailey  Macrozamia platyrachis
#> 2  2683551  Macrozamia platyrhachis F.M.Bailey Macrozamia platyrhachis
#>      rank   status confidence matchType synonym
#> 1 SPECIES ACCEPTED        100     EXACT   FALSE
#> 2 SPECIES ACCEPTED        100     EXACT   FALSE
```

* M. platyrachis w/ 4 occurrences - from GRIN Taxonomy
* M. platyr<b>h</b>achis w/ 62 occurrences - from Catalogue of Life

## Cycas circinalis vs. C. circinnalis


```r
(c3 <- name_rbind("Cycas circinalis", "Cycas circinnalis"))
#>   usageKey       scientificName     canonicalName    rank   status
#> 1  2683264  Cycas circinalis L.  Cycas circinalis SPECIES ACCEPTED
#> 2  3594916 Cycas circinnalis L. Cycas circinnalis SPECIES ACCEPTED
#>   confidence matchType synonym
#> 1         99     EXACT   FALSE
#> 2        100     EXACT   FALSE
```

* C. circinalis w/ 524 occurrences - from Catalogue of Life
* C. circin<b>n</b>alis w/ 13 occurrences - from International Plant Names Index

## Isolona perrieri vs. I. perrierii


```r
(c4 <- name_rbind("Isolona perrieri", "Isolona perrierii"))
#>   usageKey          scientificName     canonicalName    rank   status
#> 1  3648546  Isolona perrieri Diels  Isolona perrieri SPECIES ACCEPTED
#> 2  6308376 Isolona perrierii Diels Isolona perrierii SPECIES ACCEPTED
#>   confidence matchType synonym
#> 1        100     EXACT   FALSE
#> 2        100     EXACT   FALSE
```

* I. perrieri w/ 49 occurrences - from The Plant List with literature
* I. perrieri<b>i</b> w/ 30 occurrences - from Catalogue of Life

## Wiesneria vs. Wisneria


```r
(c5 <- name_rbind("Wiesneria", "Wisneria", rank = "genus"))
#>   usageKey                                                 scientificName
#> 1  2864604                                              Wiesneria Micheli
#> 2  7327444 Wisneria Micheli in Alph. de Candolle & A.C. de Candolle, 1881
#>   canonicalName  rank   status confidence matchType synonym
#> 1     Wiesneria GENUS ACCEPTED         95     EXACT   FALSE
#> 2      Wisneria GENUS ACCEPTED         95     EXACT   FALSE
```

* Wi<b>e</b>sneria w/ 71 occurrences - from Catalogue of Life
* Wisneria w/ 3 occurrences - from Interim Register of Marine and Nonmarine Genera

## The take away messages from this vignette

* Make sure you are using the name you think you're using
* Realize that GBIF's backbone taxonomy is used for occurrence data
* Searching for occurrences by name matches against backbone names, 
not other names (e.g., synonyms)
* GBIF may at some points in time have multiple version of the same name in their own backbone taxonomy - These can usually be separated by data provider (e.g., Catalogue of Life vs. International Plant Names Index)
* There are different ways to search for names - make sure are familiar 
with the four different name search functions, all starting with 
`name_`