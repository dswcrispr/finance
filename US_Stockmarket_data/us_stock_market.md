US Stock Market data
================

In this page, we make dataset of US stock market using quantmod library and NASDAQ site.

Special thanks to following refernece. <https://github.com/hyunyulhenry/HenryQuant/blob/master/R/get_US_ticker.R>

Initial settings
----------------

``` r
rm(list=ls())
options(warn = -1) # suppressing warning message
options(install.packages.check.source = "no")

# Load multiple required packages at once
packages = c('quantmod')
suppressMessages(lapply(packages, require, character.only = T, quietly = TRUE)) 
```

    ## [[1]]
    ## [1] TRUE

``` r
## Character.only = T means elements in 'packages' can be assumed to be character stirngs.
```

First, we need to get symbols of all listed issues in US stock market from NASDAQ site. (NYSE, NASDAQ, AMEX market)

we need to make function for downloading Symbol, name, sector and end prices of all listed issues.

Make 'get\_US\_symbol' function
-------------------------------

``` r
# This code is came from 'HenryQuant github'

get_US_symbol = function() {
  
  ## Following urls lead us to download each stock market's data
  url_NYSE = "http://www.nasdaq.com/screening/companies-by-name.aspx?letter=0&exchange=nyse&render=download"
  url_NASDAQ = "http://www.nasdaq.com/screening/companies-by-name.aspx?letter=0&exchange=nasdaq&render=download"
  url_AMEX = "http://www.nasdaq.com/screening/companies-by-name.aspx?letter=0&exchange=amexe&render=download"
  
  ## Download each file as csv type
  download.file(url_NYSE, destfile = "./url_NYSE.csv")
  download.file(url_NASDAQ, destfile = "./url_NASDAQ.csv")
  download.file(url_AMEX, destfile = "./url_AMEX.csv")
  
  ## Load each file
  NYSE = read.csv("./url_NYSE.csv", stringsAsFactors = F) ## 'stringsAsFactors = F' prevents columns with strings from being converted to factors
  NASDAQ = read.csv("./url_NASDAQ.csv", stringsAsFactors = F)
  AMEX = read.csv("./url_AMEX.csv", stringsAsFactors = F)
  
  
  ## Combine listed issues from three markets at one file
  us_symbol = rbind(NYSE, NASDAQ, AMEX)

  ## Data cleansing process
  us_symbol = us_symbol[us_symbol$MarketCap != "n/a", ]
  us_symbol = us_symbol[us_symbol$Sector != "n/a", ] 
  us_symbol = us_symbol[!duplicated(us_symbol$Name), ] # delete duplicated rows with respect to 'Name' 
  
  # Remove rownames of 'us_symbol'
  rownames(us_symbol) = NULL
  
  # Remove empty space in 'Symbol' using 'gsub(global substitute)' function
  us_symbol$Symbol = gsub(" ", "", us_symbol$Symbol)
  ## gsub function replaces "  " to "" in Symbol column
  
  # Remove 'IPO year' column
  us_symbol = us_symbol[, -c(5,8,9)]

  ## Saving results in csv file
  write.csv(us_symbol, "us_symbol.csv")
  
  
  ## Remove files
  file.remove("./url_NYSE.csv")
  file.remove("./url_NASDAQ.csv")
  file.remove("./url_AMEX.csv")
  
}
```

Implements 'get\_US\_symbol' function and load csv file
-------------------------------------------------------

``` r
get_US_symbol()
```

    ## [1] TRUE

``` r
us_stock = read.csv("us_symbol.csv", row.names = 1, stringsAsFactors = F)

# Show first 5 data
head(us_stock)
```

    ##   Symbol                   Name LastSale MarketCap            Sector
    ## 1    DDD 3D Systems Corporation    12.14    $1.39B        Technology
    ## 2    MMM             3M Company   207.49  $119.47B       Health Care
    ## 3   WBAI        500.com Limited    15.69   $666.6M Consumer Services
    ## 4   WUBA            58.com Inc.    62.13     $9.2B        Technology
    ## 5   EGHT                8x8 Inc    19.52    $1.87B        Technology
    ## 6    AHC  A.H. Belo Corporation     3.99   $86.35M Consumer Services
    ##                                          industry
    ## 1         Computer Software: Prepackaged Software
    ## 2                      Medical/Dental Instruments
    ## 3           Services-Misc. Amusement & Recreation
    ## 4 Computer Software: Programming, Data Processing
    ## 5                                    EDP Services
    ## 6                            Newspapers/Magazines

Now, using 'quantmod' library we make dataset containing financial indexes for each stock.

Use 'quantmod' package
----------------------

``` r
# Set various interested financial indexs in'indexes' vector with 'yahooQF' function
indexes = yahooQF(c("Price/Book", "P/E Ratio", "Price/EPS Estimate Next Year",
                             "Earning/Share", "Dividend Yield", 
                    "Market Capitalization", "Shares Outstanding", "Book Value",
                    "Earnings End Time", "Earnings Start Time"))
## We can see which indexes can be downloaded from 'getQuote' function through run yahooQF()

# Concatenate all symbols from us_stock data column 1 
codes = paste(us_stock[, 1], sep = "", collapse = ";") 

# Using 'getQuote' function to download financial indexes. 'indexes' and 'codes' are uese as arguments. 
Financial_indexes = getQuote(codes, what = indexes)
```

    ## downloading set: 1 , 2 , 3 , 4 , 5 , 6 , 7 , 8 , 9 , 10 , 11 , 12 , 13 , 14 , 15 , 16 , 17 , 18 , 19 , 20 , 21 , 22 , 23 , ...done

``` r
# Show first 5 results
head(Financial_indexes)
```

    ##               Trade Time Price/Book P/E Ratio Price/EPS Estimate Next Year
    ## DDD  2019-03-01 16:00:40  2.2598660        NA                    37.937504
    ## MMM  2019-03-01 16:02:45 12.2124790  23.33971                    17.948961
    ## WBAI 2019-03-01 16:02:05  3.2277308        NA                    60.346153
    ## WUBA 2019-03-01 16:00:07  2.8194773  30.60591                    19.537735
    ## EGHT 2019-03-01 16:01:07  8.1982360        NA                  -130.133330
    ## AHC  2019-03-01 16:02:01  0.9937734  12.23926                     8.866667
    ##      Dividend Yield Market Capitalization Shares Outstanding Book Value
    ## DDD              NA            1386157440          114181000      5.372
    ## MMM      0.02623077          119472742400          575800000     16.990
    ## WBAI             NA             667146624           42520500      4.861
    ## WUBA             NA            9203876864          148139008     22.036
    ## EGHT             NA            1867179904           95654704      2.381
    ## AHC      0.07901234              86354376           19173200      4.015
    ##      Earnings End Time Earnings Start Time
    ## DDD         1552939200          1552420800
    ## MMM         1524832200          1524486600
    ## WBAI        1495456200          1495110600
    ## WUBA        1552311000          1551879000
    ## EGHT        1559044800          1558522740
    ## AHC         1533303000          1532957400

``` r
# Merge 'financial_indexes' with exisiting 'us_stock' dataset
us_stock = cbind(us_stock[, -4], Financial_indexes[, -1])


# Show first 5 results
head(us_stock)
```

    ##   Symbol                   Name LastSale            Sector
    ## 1    DDD 3D Systems Corporation    12.14        Technology
    ## 2    MMM             3M Company   207.49       Health Care
    ## 3   WBAI        500.com Limited    15.69 Consumer Services
    ## 4   WUBA            58.com Inc.    62.13        Technology
    ## 5   EGHT                8x8 Inc    19.52        Technology
    ## 6    AHC  A.H. Belo Corporation     3.99 Consumer Services
    ##                                          industry Price/Book P/E Ratio
    ## 1         Computer Software: Prepackaged Software  2.2598660        NA
    ## 2                      Medical/Dental Instruments 12.2124790  23.33971
    ## 3           Services-Misc. Amusement & Recreation  3.2277308        NA
    ## 4 Computer Software: Programming, Data Processing  2.8194773  30.60591
    ## 5                                    EDP Services  8.1982360        NA
    ## 6                            Newspapers/Magazines  0.9937734  12.23926
    ##   Price/EPS Estimate Next Year Dividend Yield Market Capitalization
    ## 1                    37.937504             NA            1386157440
    ## 2                    17.948961     0.02623077          119472742400
    ## 3                    60.346153             NA             667146624
    ## 4                    19.537735             NA            9203876864
    ## 5                  -130.133330             NA            1867179904
    ## 6                     8.866667     0.07901234              86354376
    ##   Shares Outstanding Book Value Earnings End Time Earnings Start Time
    ## 1          114181000      5.372        1552939200          1552420800
    ## 2          575800000     16.990        1524832200          1524486600
    ## 3           42520500      4.861        1495456200          1495110600
    ## 4          148139008     22.036        1552311000          1551879000
    ## 5           95654704      2.381        1559044800          1558522740
    ## 6           19173200      4.015        1533303000          1532957400

Save results
------------

``` r
# Saving result as csv file
write.csv(us_stock, "us_stock.csv")
```
