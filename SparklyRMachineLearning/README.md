<!-- README.md is generated from README.Rmd. Please edit that file -->
Materials for:

#### [Modeling big data with R, sparklyr, and Apache Spark](https://conferences.oreilly.com/strata/strata-ca/public/schedule/detail/55791)

    Thursday Nov 2 2017,
    2:00 PM,
    Room T2,
    "Modeling big data with R, Sparklyr, and Apache Spark""
    Workshop/Training intermediate, 4 hours,
    by Dr. John Mount ( [link](https://odsc.com/training/portfolio/succeeding-big-data-real-world) ).

We have a short video showing how to install [Spark](http://spark.apache.org) using [R](https://cran.r-project.org) and [RStudio](https://www.rstudio.com) [here](https://youtu.be/qnINvPqcRvE).

Also please click through for slides from Edgar Ruiz's excellent [Strata Sparklyr presentation](https://conferences.oreilly.com/strata/strata-ca/public/schedule/detail/55800) and [cheat-sheet](http://spark.rstudio.com/images/sparklyr-cheatsheet.pdf).

#### Who is this presentation for?

Data scientists, data analysts, modelers, R users, Spark users, statisticians, and those in IT

#### Prerequisite knowledge

##### Basic familiarity with R

Experience using the [dplyr](https://CRAN.R-project.org/package=dplyr) R package (If you have not used dplyr before, please read this chapter before coming to class.) Materials or downloads needed in advance.

A WiFi-enabled laptop (You'll be provided an [RStudio Server Pro](https://www.rstudio.com/products/rstudio-server-pro/) login for students to use on the day of the workshop.)

##### What you'll learn

Learn how to quickly set up a local Spark instance, store big data in Spark and then connect to the data with R, use R to apply machine-learning algorithms to big data stored in Spark, and filter and aggregate big data stored in Spark and then import the results into R for analysis and visualization Understand how to extend R and use [sparkly](http://spark.rstudio.com)) to access the entire Spark API

### Description

Sparklyr, developed by RStudio in conjunction with IBM, Cloudera, and [H2O](http://www.h2o.ai), provides an R interface to Spark’s distributed machine-learning algorithms and much more. Sparklyr makes practical machine learning scalable and easy. With sparklyr, you can interactively manipulate Spark data using both dplyr and SQL (via DBI); filter and aggregate Spark datasets then bring them into R for analysis and visualization; orchestrate distributed machine learning from R using either Spark MLlib or H2O SparkingWater; create extensions that call the full Spark API and provide interfaces to Spark packages; and establish Spark connections and browse Spark data frames within the RStudio IDE.

John Mount demonstrates how to use sparklyr to analyze big data in Spark, covering filtering and manipulating Spark data to import into R and using R to run machine-learning algorithms on data in Spark. John also also explores the sparklyr integration built into the RStudio IDE.

Derived from ["Modeling big data with R, sparklyr, and Apache Spark" Strata Hadoop 2017]().

Public repository is: [https://github.com/WinVector/BigDataRStrata2017](https://github.com/WinVector/ODSCWest2017/tree/master/SparklyRMachineLearning).

###### config

``` r
# often a good idea, though try "n" to build source
update.packages(ask = FALSE, checkBuilt = TRUE)
```

Current list of CRAN packages used:

``` r
cranpkgs <- c(
  'DBI', 'RCurl', 'RSQLite', 'WVPlots', 'babynames', 'caret', 'cdata',
  'dbplyr', 'devtools', 'dplyr', 'dygraphs', 'e1071', 'formatR',
  'ggplot2',  'jsonlite', 'lubridate', 
  'nycflights13', 'plotly', 'rbokeh', 'replyr', 'sigr',
  'sparklyr', 'statmod', 'tidyr', 'tidyverse', 'titanic',
  'xtable' )
install.packages(cranpkgs)
sparklyr::spark_install(version = "2.2.0", hadoop_version = "2.7")
```

``` r
devpkgs <- c(
  'RStudio/EDAWR' )
for(pkgi in devpkgs) {
  devtools::install_github(pkgi)
}
```

Note: please note using `dplyr::compute()` (or `sparklyr::sdf_checkpoint()`) with `sparklyr` can have issues (see [sparklyr issue 721](https://github.com/rstudio/sparklyr/issues/721)).
