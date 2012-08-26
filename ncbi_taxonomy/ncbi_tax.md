I wrote two small functions to get the taxonomic hierarchy from the [NCBI taxonomy browser](http://www.ncbi.nlm.nih.gov/Taxonomy/Browser/wwwtax.cgi):


```r
# Get Unique ID from NCBI for give taxon-name
get_uid <- function(x) {
    x <- gsub(" ", "+", x)
    searchurl <- paste("http://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=taxonomy&term=", 
        x, sep = "")
    tt <- getURL(searchurl)
    ttp <- xmlTreeParse(tt, useInternalNodes = TRUE)
    res <- xpathSApply(ttp, "//eSearchResult/IdList/Id", xmlValue)
    # if xpath is not found return NA
    if (length(res) == 0) {
        out <- NA
    } else {
        out <- res
    }
    # NCBI limits requests to three per second
    Sys.sleep(0.33)
    return(out)
}
```



```r
# Get taxonomic hierarchy from NCBI for given UID.
get_classification <- function(x) {
    baseurl <- "http://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=taxonomy"
    ID <- paste("ID=", x, sep = "")
    searchurl <- paste(baseurl, ID, sep = "&")
    tt <- getURL(searchurl)
    ttp <- xmlTreeParse(tt, useInternalNodes = TRUE)
    out <- data.frame(ScientificName = xpathSApply(ttp, "//TaxaSet/Taxon/LineageEx/Taxon/ScientificName", 
        xmlValue), Rank = xpathSApply(ttp, "//TaxaSet/Taxon/LineageEx/Taxon/Rank", 
        xmlValue), UID = xpathSApply(ttp, "//TaxaSet/Taxon/LineageEx/Taxon/TaxId", 
        xmlValue))
    # NCBI limits requests to three per second
    Sys.sleep(0.33)
    return(out)
}
```


Here is an example: 

```r
require(RCurl)
require(XML)
```


```r
uid <- get_uid("Hydropsyche angustipennis")
# Unique ID in NCBI
uid
```

```
## [1] "329908"
```

```r
# hierarchial classification
get_classification(uid)
```

```
##        ScientificName         Rank     UID
## 1  cellular organisms      no rank  131567
## 2           Eukaryota superkingdom    2759
## 3        Opisthokonta      no rank   33154
## 4             Metazoa      kingdom   33208
## 5           Eumetazoa      no rank    6072
## 6           Bilateria      no rank   33213
## 7           Coelomata      no rank   33316
## 8         Protostomia      no rank   33317
## 9           Ecdysozoa      no rank 1206794
## 10      Panarthropoda      no rank   88770
## 11         Arthropoda       phylum    6656
## 12        Mandibulata      no rank  197563
## 13       Pancrustacea      no rank  197562
## 14           Hexapoda   superclass    6960
## 15            Insecta        class   50557
## 16         Dicondylia      no rank   85512
## 17          Pterygota      no rank    7496
## 18           Neoptera     subclass   33340
## 19      Endopterygota   infraclass   33392
## 20   Amphiesmenoptera   superorder   85604
## 21        Trichoptera        order   30263
## 22       Annulipalpia     suborder   93873
## 23    Hydropsychoidea  superfamily   41029
## 24     Hydropsychidae       family   41030
## 25     Hydropsychinae    subfamily  147297
## 26        Hydropsyche        genus   50443
```




Also have a look at the [rOpenSci-project](http://ropensci.org) and their [packages](https://github.com/ropensci).
[![alt text](http://assets.ropensci.org/media_kit/ropensci_main.png)](http://ropensci.org)



