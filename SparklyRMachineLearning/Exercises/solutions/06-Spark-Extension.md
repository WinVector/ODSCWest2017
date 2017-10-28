This is only a concept script, it runs correctly but is intended for teaching not direct use in production.

At some point the `SparkR:dapply()` functionality we are working to capture here will be available as a method called [`sparklyr::spark_apply()`](https://github.com/rstudio/sparklyr/pull/728).

One point in particular is this script assumes none of the following directories are present (as it is going to try to create them and write its own temp results):

-   Exercises/solutions/df\*\_tmp
-   Exercises/solutions/tmpFile\*\_\*

We don't delete these as we don't want to perform too many (potentially unsafe) file operations on the user's behalf.

``` r
Sys.setenv(TZ='UTC')
suppressPackageStartupMessages(library("sparklyr"))
packageVersion("sparklyr")
```

    ## [1] '0.6.3'

``` r
suppressPackageStartupMessages(library("dplyr"))
packageVersion("dplyr")
```

    ## [1] '0.7.4'

``` r
sc <- spark_connect(master = "local")
```

    ## * Using Spark: 2.2.0

``` r
d <- data_frame(x = c(20100101120101, "2009-01-02 12-01-02", "2009.01.03 12:01:03",
       "2009-1-4 12-1-4",
       "2009-1, 5 12:1, 5",
       "200901-08 1201-08",
       "2009 arbitrary 1 non-decimal 6 chars 12 in between 1 !!! 6",
       "OR collapsed formats: 20090107 120107 (as long as prefixed with zeros)",
       "Automatic wday, Thu, detection, 10-01-10 10:01:10 and p format: AM",
       "Created on 10-01-11 at 10:01:11 PM"))

df  <- copy_to(sc, d, 'df', 
               temporary = TRUE, overwrite = TRUE)
print(df)
```

    ## # Source:   table<df> [?? x 1]
    ## # Database: spark_connection
    ##                                                                         x
    ##                                                                     <chr>
    ##  1                                                         20100101120101
    ##  2                                                    2009-01-02 12-01-02
    ##  3                                                    2009.01.03 12:01:03
    ##  4                                                        2009-1-4 12-1-4
    ##  5                                                      2009-1, 5 12:1, 5
    ##  6                                                      200901-08 1201-08
    ##  7             2009 arbitrary 1 non-decimal 6 chars 12 in between 1 !!! 6
    ##  8 OR collapsed formats: 20090107 120107 (as long as prefixed with zeros)
    ##  9     Automatic wday, Thu, detection, 10-01-10 10:01:10 and p format: AM
    ## 10                                     Created on 10-01-11 at 10:01:11 PM
    ## # ... with more rows

Running `SQL` directly (see <http://spark.rstudio.com>).

``` r
library("DBI")

# returns a in-memor data.frame
dfx <- dbGetQuery(sc, "SELECT * FROM df LIMIT 5")
dfx
```

    ##                     x
    ## 1      20100101120101
    ## 2 2009-01-02 12-01-02
    ## 3 2009.01.03 12:01:03
    ## 4     2009-1-4 12-1-4
    ## 5   2009-1, 5 12:1, 5

``` r
# build another table
dbSendQuery(sc, "CREATE TABLE df2 AS SELECT * FROM df LIMIT 5")
```

    ## <DBISparkResult>
    ##   SQL  CREATE TABLE df2 AS SELECT * FROM df LIMIT 5
    ##   ROWS Fetched: 0 [complete]
    ##        Changed: 0

``` r
# get a handle to it
dbListTables(sc)
```

    ## [1] "df"  "df2"

``` r
df2 <- dplyr::tbl(sc, 'df2')
df2
```

    ## # Source:   table<df2> [?? x 1]
    ## # Database: spark_connection
    ##                     x
    ##                 <chr>
    ## 1      20100101120101
    ## 2 2009-01-02 12-01-02
    ## 3 2009.01.03 12:01:03
    ## 4     2009-1-4 12-1-4
    ## 5   2009-1, 5 12:1, 5

Using `SparkR` for `R` user defined functions. Replaced by [`sparklyr::spark_apply()`](https://spark.rstudio.com/articles/guides-distributed-r.html) ([wasn't available until Sparklyr 0.6.0](https://github.com/rstudio/sparklyr/blob/master/NEWS.md), [released to CRAN 29-Jul-2017](https://cran.rstudio.com/src/contrib/Archive/sparklyr/)).

The following doesn't always run in a knitr evironment. And using `SparkR` in production would entail already having the needed R packages installed.

``` r
dMany  <- copy_to(sc, bind_rows(rep(list(d), 100000)), 'dMany', 
                  temporary = TRUE, overwrite = TRUE)
f <- function(df, ...) {
  df$cleaned = as.character(lubridate::ymd_hms(df$x))
  df$nrow <- nrow(df)
  df$clsstr <- paste(class(df), collapse = ' ')
  df
}
dfR <- spark_apply(dMany, f, 
                   columns = c(colnames(dMany), 'cleaned', 'nrow', 'clsstr'))
replyr::replyr_nrow(dMany)
```

    ## [1] 1e+06

``` r
replyr::replyr_nrow(dfR)
```

    ## [1] 1e+06

``` r
class(dfR)
```

    ## [1] "tbl_spark" "tbl_sql"   "tbl_lazy"  "tbl"

``` r
glimpse(dfR)
```

    ## Observations: 25
    ## Variables: 4
    ## $ x       <chr> "20100101120101", "2009-01-02 12-01-02", "2009.01.03 1...
    ## $ cleaned <chr> "2010-01-01 12:01:01", "2009-01-02 12:01:02", "2009-01...
    ## $ nrow    <int> 279209, 279209, 279209, 279209, 279209, 279209, 279209...
    ## $ clsstr  <chr> "data.frame", "data.frame", "data.frame", "data.frame"...

``` r
print(dfR)
```

    ## # Source:   table<sparklyr_tmp_e2a978c4db8f> [?? x 4]
    ## # Database: spark_connection
    ##                                                                         x
    ##                                                                     <chr>
    ##  1                                                         20100101120101
    ##  2                                                    2009-01-02 12-01-02
    ##  3                                                    2009.01.03 12:01:03
    ##  4                                                        2009-1-4 12-1-4
    ##  5                                                      2009-1, 5 12:1, 5
    ##  6                                                      200901-08 1201-08
    ##  7             2009 arbitrary 1 non-decimal 6 chars 12 in between 1 !!! 6
    ##  8 OR collapsed formats: 20090107 120107 (as long as prefixed with zeros)
    ##  9     Automatic wday, Thu, detection, 10-01-10 10:01:10 and p format: AM
    ## 10                                     Created on 10-01-11 at 10:01:11 PM
    ## # ... with more rows, and 3 more variables: cleaned <chr>, nrow <int>,
    ## #   clsstr <chr>

From: <http://spark.rstudio.com/extensions.html>.

``` r
count_lines <- function(sc, file) {
  spark_context(sc) %>% 
    invoke("textFile", file, 1L) %>% 
    invoke("count")
}

count_lines(sc, "tmp.csv")
```

    ## [1] 3

A simple Java example.

``` r
billionBigInteger <- invoke_new(sc, "java.math.BigInteger", "1000000000")
print(billionBigInteger)
```

    ## <jobj[175]>
    ##   class java.math.BigInteger
    ##   1000000000

``` r
str(billionBigInteger)
```

    ## Classes 'spark_jobj', 'shell_jobj' <environment: 0x7fbead9b5d80>

``` r
billion <- invoke(billionBigInteger, "longValue")
str(billion)
```

    ##  num 1e+09

``` r
spark_disconnect(sc)
```
