# R with Big Data 3b: Big Data and Spark SQL 2/2
Garrett Grolemund, Nathan Stephens, John Mount and Nina Zumel  

Use dplyr syntax to write Apache Spark SQL queries. Use select, where, group by, joins, and window functions in Aparche Spark SQL.

## Setup


```r
knitr::opts_chunk$set(warning = FALSE, message = FALSE)
library(sparklyr)
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(tidyr)
library(babynames)
library(ggplot2)
library(dygraphs)
library(rbokeh)
```

## Connect to Spark

Install and connect to a local Spark instance. Copy data into Spark DataFrames.


```r
sc <- spark_connect(master = "local")
babynames_tbl <- copy_to(sc, babynames, "babynames", overwrite = TRUE)
applicants_tbl <- copy_to(sc, applicants, "applicants", overwrite = TRUE)
```

## Total US births

Plot total US births recorded from the Social Security Administration.

First, for practice, do it locally in the applicants data frame, and tidyr::spread


```r
# local approach
birthsYearly <- applicants %>%  
  mutate(n_all = n_all/1000000) %>%
  tidyr::spread(sex, n_all)

# try it with remote data. Oops! This fails at tidyr::spread
birthsYearly <- applicants_tbl %>%  
  mutate(n_all = n_all/1000000) %>%
  tidyr::spread(sex, n_all)
```

```
## Error in UseMethod("spread_"): no applicable method for 'spread_' applied to an object of class "c('tbl_spark', 'tbl_sql', 'tbl_lazy', 'tbl')"
```

Now simulate `tidyr::spread` on the remote data, and continue.


```r
# by-hand spread on remote data
birthsYearly <- applicants_tbl %>%
  mutate(male = ifelse(sex == "M", n_all, 0), female = ifelse(sex == "F", n_all, 0)) %>%
  group_by(year) %>%
  summarize(Male = sum(male) / 1000000, Female = sum(female) / 1000000) %>%
  arrange(year) %>%
  collect

# ## Or use dev-version of replyr: https://github.com/WinVector/replyr
# birthsYearly <- applicants_tbl %>%
#   replyr::replyr_moveValuesToColumns(columnToTakeKeysFrom = 'sex',
#                                      columnToTakeValuesFrom = 'n_all',
#                                      rowKeyColumns = 'year') %>%
#   mutate(Male= M/1000000, Female = F/1000000) %>% 
#   select(year, Male, Female) %>%
#   arrange(year) %>% 
#   collect

# plot it
birthsYearly %>%
  dygraph(main = "Total US Births (SSN)", ylab = "Millions") %>%
  dySeries("Female") %>%
  dySeries("Male") %>%
  dyOptions(stackedGraph = TRUE) %>%
  dyRangeSelector(height = 20) 
```

<!--html_preserve--><div id="htmlwidget-0c0c60ccd49e4263c17c" style="width:672px;height:480px;" class="dygraphs html-widget"></div>
<script type="application/json" data-for="htmlwidget-0c0c60ccd49e4263c17c">{"x":{"attrs":{"title":"Total US Births (SSN)","ylabel":"Millions","labels":["year","Female","Male"],"legend":"auto","retainDateWindow":false,"axes":{"x":{"pixelsPerLabel":60,"drawAxis":true},"y":{"drawAxis":true}},"series":{"Female":{"axis":"y"},"Male":{"axis":"y"}},"stackedGraph":true,"fillGraph":false,"fillAlpha":0.15,"stepPlot":false,"drawPoints":false,"pointSize":1,"drawGapEdgePoints":false,"connectSeparatedPoints":false,"strokeWidth":1,"strokeBorderColor":"white","colorValue":0.5,"colorSaturation":1,"includeZero":false,"drawAxesAtZero":false,"logscale":false,"axisTickSize":3,"axisLineColor":"black","axisLineWidth":0.3,"axisLabelColor":"black","axisLabelFontSize":14,"axisLabelWidth":60,"drawGrid":true,"gridLineWidth":0.3,"rightGap":5,"digitsAfterDecimal":2,"labelsKMB":false,"labelsKMG2":false,"labelsUTC":false,"maxNumberWidth":6,"animatedZooms":false,"mobileDisableYTouch":true,"showRangeSelector":true,"rangeSelectorHeight":20,"rangeSelectorPlotFillColor":" #A7B1C4","rangeSelectorPlotStrokeColor":"#808FAB","interactionModel":"Dygraph.Interaction.defaultModel"},"annotations":[],"shadings":[],"events":[],"format":"numeric","data":[[1880,1881,1882,1883,1884,1885,1886,1887,1888,1889,1890,1891,1892,1893,1894,1895,1896,1897,1898,1899,1900,1901,1902,1903,1904,1905,1906,1907,1908,1909,1910,1911,1912,1913,1914,1915,1916,1917,1918,1919,1920,1921,1922,1923,1924,1925,1926,1927,1928,1929,1930,1931,1932,1933,1934,1935,1936,1937,1938,1939,1940,1941,1942,1943,1944,1945,1946,1947,1948,1949,1950,1951,1952,1953,1954,1955,1956,1957,1958,1959,1960,1961,1962,1963,1964,1965,1966,1967,1968,1969,1970,1971,1972,1973,1974,1975,1976,1977,1978,1979,1980,1981,1982,1983,1984,1985,1986,1987,1988,1989,1990,1991,1992,1993,1994,1995,1996,1997,1998,1999,2000,2001,2002,2003,2004,2005,2006,2007,2008,2009,2010,2011,2012,2013,2014,2015],[0.097604,0.098855,0.115696,0.120059,0.137586,0.141949,0.153736,0.155422,0.189447,0.189219,0.201661,0.196567,0.224915,0.225232,0.235972,0.247107,0.251993,0.248275,0.274146,0.24749,0.317775,0.254232,0.280333,0.278198,0.292438,0.30987,0.313441,0.337433,0.354533,0.368098,0.419526,0.441812,0.586708,0.654904,0.796608,1.023882,1.085708,1.123702,1.202363,1.17465,1.244038,1.279681,1.247531,1.252416,1.295716,1.263046,1.230117,1.236348,1.195385,1.157481,1.166344,1.103563,1.106167,1.045864,1.082174,1.086677,1.077429,1.101723,1.141329,1.133993,1.181212,1.245808,1.390365,1.435227,1.366429,1.346051,1.612831,1.817781,1.742587,1.755499,1.758923,1.846667,1.902145,1.929114,1.990632,2.004373,2.05933,2.097516,2.065011,2.078477,2.07993,2.076225,2.027018,1.98789,1.957182,1.827342,1.755555,1.716654,1.70948,1.76272,1.831956,1.752356,1.612524,1.554011,1.566158,1.560705,1.571901,1.644879,1.643759,1.723064,1.780228,1.788032,1.813716,1.789105,1.802581,1.845612,1.844734,1.873574,1.922263,1.991787,2.053702,2.032978,2.004166,1.971029,1.948875,1.921035,1.916678,1.908561,1.937701,1.945832,1.994461,1.979571,1.973556,2.004932,2.016063,2.027508,2.088261,2.113906,2.079948,2.021799,1.956853,1.933342,1.934051,1.920953,1.947553,1.935109],[0.118399,0.108282,0.122031,0.112478,0.122739,0.115946,0.119042,0.109315,0.129905,0.119034,0.119701,0.109267,0.131453,0.12104,0.124894,0.126644,0.129072,0.121943,0.132105,0.115195,0.16214,0.115595,0.132749,0.129326,0.138508,0.143244,0.14407,0.158588,0.166372,0.176868,0.208524,0.241392,0.451456,0.536243,0.683321,0.880948,0.923258,0.959331,1.048689,1.015339,1.100869,1.137925,1.125316,1.132363,1.169028,1.151452,1.145465,1.161789,1.141098,1.107405,1.129373,1.069368,1.074167,1.019889,1.061656,1.069275,1.064111,1.093413,1.136223,1.133089,1.185984,1.254586,1.408028,1.454263,1.388905,1.371329,1.650135,1.857349,1.78257,1.801901,1.819993,1.910588,1.973397,2.001102,2.067882,2.088636,2.14468,2.187626,2.153177,2.166434,2.166091,2.15591,2.102338,2.065419,2.027616,1.895639,1.818163,1.780001,1.776423,1.830456,1.906003,1.81863,1.674919,1.61437,1.630935,1.623186,1.633331,1.709885,1.708989,1.791697,1.854692,1.86208,1.886704,1.862811,1.875736,1.923356,1.920483,1.949062,2.000862,2.095209,2.150863,2.118988,2.098423,2.064806,2.037777,2.010773,2.003109,1.997063,2.026806,2.03792,2.086976,2.066991,2.065175,2.099617,2.111689,2.12538,2.190066,2.212515,2.177409,2.117746,2.050778,2.026895,2.023569,2.013559,2.039794,2.026872]],"fixedtz":false,"tzone":""},"evals":["attrs.interactionModel"],"jsHooks":[]}</script><!--/html_preserve-->

## Aggregate data by name

Use Spark SQL to create a look up table. 


```r
topNames_tbl <- babynames_tbl %>%
  filter(year >= 1986) %>%  
  group_by(name, sex) %>%
  summarize(count = sum(n)) %>%
  filter(count > 1000) %>%
  select(name, sex) %>%
  compute

yearlyNames_tbl <- babynames_tbl %>%
  filter(year >= 1986) %>%
  inner_join(topNames_tbl, by = c("sex", "name")) %>%
  compute
```

## Most popular names (1986)

Identify the top 5 male and female names from 1986. Visualize the popularity trend over time.


```r
topNames1986_tbl <- yearlyNames_tbl %>%
  filter(year == 1986) %>%
  group_by(sex) %>%
  mutate(rank = min_rank(desc(n))) %>%
  filter(rank <= 5) %>%
  arrange(sex, rank) %>%
  select(name, sex, rank) %>%
  compute


topNames1986Yearly <- yearlyNames_tbl %>%
  inner_join(topNames1986_tbl, by = c("sex", "name")) %>%
  collect

ggplot(topNames1986Yearly, aes(year, n, color=name)) +
  facet_grid(~sex) +
  geom_line() +
  ggtitle("Most Popular Names of 1986")
```

![](03b-Spark-SQL_files/figure-html/unnamed-chunk-5-1.png)<!-- -->



## Shared names

Visualize the most popular names that are shared by both males and females.


```r
sharedName <- babynames_tbl %>%
  mutate(male = ifelse(sex == "M", n, 0), female = ifelse(sex == "F", n, 0)) %>%
  group_by(name) %>%
  summarize(Male = as.numeric(sum(male)), 
            Female = as.numeric(sum(female)),
            count = sum(n),
            AvgYear = round(as.numeric(sum(year * n) / sum(n)),0)) %>%
  filter(Male > 30000 & Female > 30000) %>%
  collect

figure(width = NULL, height = NULL, 
       xlab = "Log10 Number of Males", 
       ylab = "Log10 Number of Females",
       title = "Top shared names (1880 - 2014)") %>%
  ly_points(log10(Male), log10(Female), data = sharedName,
            color = AvgYear, size = scale(sqrt(count)),
            hover = list(name, Male, Female, AvgYear), legend = FALSE)
```

<!--html_preserve--><div id="htmlwidget-0a83575690b4c3c66cda" style="width:672px;height:480px;" class="rbokeh html-widget"></div>
<script type="application/json" data-for="htmlwidget-0a83575690b4c3c66cda">{"x":{"elementid":"c8e06e008652d3d05ccb9c0d31ce30b2","modeltype":"Plot","modelid":"e35b95631cda6b0b3c1ef539018e63e1","docid":"ae67dd81acfe0414bc25cf3231571457","docs_json":{"ae67dd81acfe0414bc25cf3231571457":{"version":"0.12.2","title":"Bokeh Figure","roots":{"root_ids":["e35b95631cda6b0b3c1ef539018e63e1"],"references":[{"type":"Plot","id":"e35b95631cda6b0b3c1ef539018e63e1","attributes":{"id":"e35b95631cda6b0b3c1ef539018e63e1","sizing_mode":"scale_both","x_range":{"type":"Range1d","id":"8e10b3d9d152fbe91521b7c1a2f8e066"},"y_range":{"type":"Range1d","id":"f34615f42f2e8b1a331c6c34b3294eba"},"left":[{"type":"LinearAxis","id":"c8b4c3f1d024181c6a9c601d99ccbf8f"}],"below":[{"type":"LinearAxis","id":"30291e4d1e87fa1f5b24113151571a23"}],"right":[],"above":[],"renderers":[{"type":"BoxAnnotation","id":"6bebcc9a9b87a454ba1ec0929d319307"},{"type":"GlyphRenderer","id":"2beafdaca4ff7758d9dc08983fe748b9"},{"type":"LinearAxis","id":"30291e4d1e87fa1f5b24113151571a23"},{"type":"Grid","id":"48f2f92612d72ee75e2ad911a22e805b"},{"type":"LinearAxis","id":"c8b4c3f1d024181c6a9c601d99ccbf8f"},{"type":"Grid","id":"f5ae90077527a9e67e63db540d6ce4fc"}],"extra_y_ranges":{},"extra_x_ranges":{},"tags":[],"min_border_left":4,"min_border_right":4,"min_border_top":4,"min_border_bottom":4,"lod_threshold":null,"toolbar":{"type":"Toolbar","id":"2d10bd62515ce974d93dca8990ee62b3"},"tool_events":{"type":"ToolEvents","id":"21a4f6f7fd80deaecb8851d275beb258"},"title":{"type":"Title","id":"e5adb913ee1421347ad447649d3ed1f6"}},"subtype":"Figure"},{"type":"Toolbar","id":"2d10bd62515ce974d93dca8990ee62b3","attributes":{"id":"2d10bd62515ce974d93dca8990ee62b3","tags":[],"active_drag":"auto","active_scroll":"auto","active_tap":"auto","tools":[{"type":"PanTool","id":"022492b7a758069fdea2f9e5bed5e159"},{"type":"WheelZoomTool","id":"9fd5988e04785c9ec7f5c5a58f93fc68"},{"type":"BoxZoomTool","id":"decb96639eb3783e81bc97f84b871e0b"},{"type":"ResetTool","id":"de2a8cf8062f3678c847427c551e00b6"},{"type":"SaveTool","id":"59fbc0f31d51111b62fb65ceb366ec20"},{"type":"HelpTool","id":"3a18e6649b682706d269d39825f27690"},{"type":"HoverTool","id":"6d54ddd910ef9a7dff7cd568b0ecb8b3"}],"logo":null}},{"type":"PanTool","id":"022492b7a758069fdea2f9e5bed5e159","attributes":{"id":"022492b7a758069fdea2f9e5bed5e159","tags":[],"plot":{"type":"Plot","id":"e35b95631cda6b0b3c1ef539018e63e1","subtype":"Figure"},"dimensions":["width","height"]}},{"type":"ToolEvents","id":"21a4f6f7fd80deaecb8851d275beb258","attributes":{"id":"21a4f6f7fd80deaecb8851d275beb258","tags":[]},"geometries":[]},{"type":"WheelZoomTool","id":"9fd5988e04785c9ec7f5c5a58f93fc68","attributes":{"id":"9fd5988e04785c9ec7f5c5a58f93fc68","tags":[],"plot":{"type":"Plot","id":"e35b95631cda6b0b3c1ef539018e63e1","subtype":"Figure"},"dimensions":["width","height"]}},{"type":"BoxAnnotation","id":"6bebcc9a9b87a454ba1ec0929d319307","attributes":{"id":"6bebcc9a9b87a454ba1ec0929d319307","tags":[],"line_color":{"units":"data","value":"black"},"line_alpha":{"units":"data","value":1},"fill_color":{"units":"data","value":"lightgrey"},"fill_alpha":{"units":"data","value":0.5},"line_dash":[4,4],"line_width":{"units":"data","value":2},"level":"overlay","top_units":"screen","bottom_units":"screen","left_units":"screen","right_units":"screen","render_mode":"css"}},{"type":"BoxZoomTool","id":"decb96639eb3783e81bc97f84b871e0b","attributes":{"id":"decb96639eb3783e81bc97f84b871e0b","tags":[],"plot":{"type":"Plot","id":"e35b95631cda6b0b3c1ef539018e63e1","subtype":"Figure"},"overlay":{"type":"BoxAnnotation","id":"6bebcc9a9b87a454ba1ec0929d319307"}}},{"type":"ResetTool","id":"de2a8cf8062f3678c847427c551e00b6","attributes":{"id":"de2a8cf8062f3678c847427c551e00b6","tags":[],"plot":{"type":"Plot","id":"e35b95631cda6b0b3c1ef539018e63e1","subtype":"Figure"}}},{"type":"SaveTool","id":"59fbc0f31d51111b62fb65ceb366ec20","attributes":{"id":"59fbc0f31d51111b62fb65ceb366ec20","tags":[],"plot":{"type":"Plot","id":"e35b95631cda6b0b3c1ef539018e63e1","subtype":"Figure"}}},{"type":"HelpTool","id":"3a18e6649b682706d269d39825f27690","attributes":{"id":"3a18e6649b682706d269d39825f27690","tags":[],"plot":{"type":"Plot","id":"e35b95631cda6b0b3c1ef539018e63e1","subtype":"Figure"},"redirect":"http://hafen.github.io/rbokeh","help_tooltip":"Click to learn more about rbokeh."}},{"type":"Title","id":"e5adb913ee1421347ad447649d3ed1f6","attributes":{"id":"e5adb913ee1421347ad447649d3ed1f6","tags":[],"plot":null,"text":"Top shared names (1880 - 2014)"}},{"type":"HoverTool","id":"6d54ddd910ef9a7dff7cd568b0ecb8b3","attributes":{"id":"6d54ddd910ef9a7dff7cd568b0ecb8b3","tags":[],"plot":{"type":"Plot","id":"e35b95631cda6b0b3c1ef539018e63e1","subtype":"Figure"},"renderers":[{"type":"GlyphRenderer","id":"2beafdaca4ff7758d9dc08983fe748b9"}],"names":[],"anchor":"center","attachment":"horizontal","line_policy":"prev","mode":"mouse","point_policy":"snap_to_data","tooltips":[["name","@hover_col_1"],["Male","@hover_col_2"],["Female","@hover_col_3"],["AvgYear","@hover_col_4"]]}},{"type":"ColumnDataSource","id":"3f197aaf671b36d808588b0a7e65dd71","attributes":{"id":"3f197aaf671b36d808588b0a7e65dd71","tags":[],"column_names":["x","y","size","line_color","fill_color","hover_col_1","hover_col_2","hover_col_3","hover_col_4"],"selected":[],"data":{"x":[4.85632400448479,5.03594977953667,4.89345659488572,5.65136622110655,4.6031985206761,4.49909570982972,4.69210631378532,4.71409494250139,4.78634674370446,4.92789879238942,4.52288743108312,4.54225262685987,4.82160522494917,4.69444724494677,5.47726888974698,5.00548509843934,4.95223529802618,5.55491416080625,4.65732361711215,5.33408766358447,5.03791231718231,5.03965202358192,5.36179391928037,5.05151139090722,4.71781191687659,4.61496045819744,4.72431683884585,5.62510451615495,4.78575679996264,4.91018668871114,4.64586419943835,4.49288605092027],"y":[5.27402615490758,5.49403069106064,4.95629812295601,5.16475127202258,4.51356383185059,5.00491006712558,5.00332263895645,5.46910004571425,5.52085151179255,5.42566950889099,4.73972262072721,5.25379580891004,4.69506118801271,4.68552677680185,4.51759173071191,4.68865114678907,4.94124799339657,5.10766438386765,4.79574797174963,4.96783335617857,4.87738286546453,5.22033825543726,4.79282576697757,5.42316884129164,5.25923062051007,5.31847238130028,5.2807854530533,4.98618481644248,5.39896026993601,5.67202115660551,5.46075656136594,4.74550418708699],"size":[12.2857142857143,17.4285714285714,7.14285714285714,20,2,7.14285714285714,7.14285714285714,14.8571428571429,14.8571428571429,14.8571428571429,4.57142857142857,9.71428571428572,4.57142857142857,4.57142857142857,14.8571428571429,7.14285714285714,7.14285714285714,17.4285714285714,4.57142857142857,12.2857142857143,7.14285714285714,12.2857142857143,12.2857142857143,14.8571428571429,9.71428571428572,9.71428571428572,9.71428571428572,20,12.2857142857143,20,14.8571428571429,4.57142857142857],"line_color":["#66C2A4","#005B24","#3CA96F","#66C2A4","#3CA96F","#50B689","#00441B","#19823D","#005B24","#05712F","#005B24","#2B9453","#05712F","#2B9453","#19823D","#50B689","#00441B","#005B24","#00441B","#005B24","#05712F","#50B689","#3CA96F","#2B9453","#3CA96F","#005B24","#19823D","#3CA96F","#2B9453","#19823D","#2B9453","#2B9453"],"fill_color":["#66C2A4","#005B24","#3CA96F","#66C2A4","#3CA96F","#50B689","#00441B","#19823D","#005B24","#05712F","#005B24","#2B9453","#05712F","#2B9453","#19823D","#50B689","#00441B","#005B24","#00441B","#005B24","#05712F","#50B689","#3CA96F","#2B9453","#3CA96F","#005B24","#19823D","#3CA96F","#2B9453","#19823D","#2B9453","#2B9453"],"hover_col_1":["Marion ","Taylor ","Jackie ","Willie ","Frankie","Billie ","Avery  ","Shannon","Alexis ","Jamie  ","Kendall","Kim    ","Jaime  ","Kerry  ","Shawn  ","Johnnie","Riley  ","Jordan ","Peyton ","Angel  ","Casey  ","Jessie ","Lee    ","Leslie ","Lynn   ","Morgan ","Dana   ","Terry  ","Tracy  ","Kelly  ","Robin  ","Jody   "],"hover_col_2":[" 71833","108630"," 78245","448091"," 40105"," 31557"," 49216"," 51772"," 61143"," 84703"," 33334"," 34854"," 66314"," 49482","300102","101271"," 89585","358851"," 45428","215818","109122","109560","230035","112593"," 52217"," 41206"," 53005","421798"," 61060"," 81318"," 44245"," 31109"],"hover_col_3":["187943","311911"," 90427","146134"," 32626","101137","100768","294510","331781","266483"," 54919","179389"," 49552"," 48476"," 32930"," 48826"," 87347","128134"," 62481"," 92861"," 75402","166088"," 62062","264953","181648","208196","190891"," 96869","250588","469917","288906"," 55655"],"hover_col_4":["1931","1997","1956","1938","1956","1944","2003","1979","1999","1981","1994","1963","1984","1969","1980","1941","2001","1997","2006","1996","1988","1942","1951","1966","1957","1995","1972","1960","1970","1976","1965","1969"]}}},{"type":"Circle","id":"ddab2516b42584d57d99c6a83be4d2e6","attributes":{"id":"ddab2516b42584d57d99c6a83be4d2e6","tags":[],"visible":true,"line_alpha":{"units":"data","value":1},"fill_alpha":{"units":"data","value":0.5},"x":{"units":"data","field":"x"},"y":{"units":"data","field":"y"},"size":{"units":"screen","field":"size"},"line_color":{"units":"data","field":"line_color"},"fill_color":{"units":"data","field":"fill_color"}}},{"type":"Circle","id":"47b6e20f303b1646d317d1ac1740ec12","attributes":{"id":"47b6e20f303b1646d317d1ac1740ec12","tags":[],"visible":true,"line_alpha":{"units":"data","value":1},"fill_alpha":{"units":"data","value":0.5},"x":{"units":"data","field":"x"},"y":{"units":"data","field":"y"},"size":{"units":"screen","field":"size"},"line_color":{"units":"data","value":"#e1e1e1"},"fill_color":{"units":"data","value":"#e1e1e1"}}},{"type":"Circle","id":"8bb68cf4a7bdcc31cca26a3b4df0191d","attributes":{"id":"8bb68cf4a7bdcc31cca26a3b4df0191d","tags":[],"visible":true,"line_alpha":{"units":"data","value":1},"fill_alpha":{"units":"data","value":1},"x":{"units":"data","field":"x"},"y":{"units":"data","field":"y"},"size":{"units":"screen","field":"size"},"line_color":{"units":"data","field":"line_color"},"fill_color":{"units":"data","field":"fill_color"}}},{"type":"GlyphRenderer","id":"2beafdaca4ff7758d9dc08983fe748b9","attributes":{"id":"2beafdaca4ff7758d9dc08983fe748b9","tags":[],"selection_glyph":null,"nonselection_glyph":{"type":"Circle","id":"47b6e20f303b1646d317d1ac1740ec12"},"hover_glyph":{"type":"Circle","id":"8bb68cf4a7bdcc31cca26a3b4df0191d"},"name":null,"data_source":{"type":"ColumnDataSource","id":"3f197aaf671b36d808588b0a7e65dd71"},"glyph":{"type":"Circle","id":"ddab2516b42584d57d99c6a83be4d2e6"}}},{"type":"Range1d","id":"8e10b3d9d152fbe91521b7c1a2f8e066","attributes":{"id":"8e10b3d9d152fbe91521b7c1a2f8e066","tags":[],"start":4.41179243900723,"end":5.73245983301959}},{"type":"Range1d","id":"f34615f42f2e8b1a331c6c34b3294eba","attributes":{"id":"f34615f42f2e8b1a331c6c34b3294eba","tags":[],"start":4.43247181911774,"end":5.75311316933836}},{"type":"LinearAxis","id":"30291e4d1e87fa1f5b24113151571a23","attributes":{"id":"30291e4d1e87fa1f5b24113151571a23","tags":[],"plot":{"type":"Plot","id":"e35b95631cda6b0b3c1ef539018e63e1","subtype":"Figure"},"axis_label":"Log10 Number of Males","formatter":{"type":"BasicTickFormatter","id":"d243759b52b9c20f6dafca326858e0c8"},"ticker":{"type":"BasicTicker","id":"bd251f59ae76dbc1b66ef425f0cccbdb"},"visible":true,"axis_label_text_font_size":"12pt"}},{"type":"BasicTickFormatter","id":"d243759b52b9c20f6dafca326858e0c8","attributes":{"id":"d243759b52b9c20f6dafca326858e0c8","tags":[]}},{"type":"BasicTicker","id":"bd251f59ae76dbc1b66ef425f0cccbdb","attributes":{"id":"bd251f59ae76dbc1b66ef425f0cccbdb","tags":[],"num_minor_ticks":5}},{"type":"Grid","id":"48f2f92612d72ee75e2ad911a22e805b","attributes":{"id":"48f2f92612d72ee75e2ad911a22e805b","tags":[],"dimension":0,"plot":{"type":"Plot","id":"e35b95631cda6b0b3c1ef539018e63e1","subtype":"Figure"},"ticker":{"type":"BasicTicker","id":"bd251f59ae76dbc1b66ef425f0cccbdb"}}},{"type":"LinearAxis","id":"c8b4c3f1d024181c6a9c601d99ccbf8f","attributes":{"id":"c8b4c3f1d024181c6a9c601d99ccbf8f","tags":[],"plot":{"type":"Plot","id":"e35b95631cda6b0b3c1ef539018e63e1","subtype":"Figure"},"axis_label":"Log10 Number of Females","formatter":{"type":"BasicTickFormatter","id":"995e3763560a3fc9a9df49aca0931443"},"ticker":{"type":"BasicTicker","id":"b70339961782cb7f9397e14c24ebe8a4"},"visible":true,"axis_label_text_font_size":"12pt"}},{"type":"BasicTickFormatter","id":"995e3763560a3fc9a9df49aca0931443","attributes":{"id":"995e3763560a3fc9a9df49aca0931443","tags":[]}},{"type":"BasicTicker","id":"b70339961782cb7f9397e14c24ebe8a4","attributes":{"id":"b70339961782cb7f9397e14c24ebe8a4","tags":[],"num_minor_ticks":5}},{"type":"Grid","id":"f5ae90077527a9e67e63db540d6ce4fc","attributes":{"id":"f5ae90077527a9e67e63db540d6ce4fc","tags":[],"dimension":1,"plot":{"type":"Plot","id":"e35b95631cda6b0b3c1ef539018e63e1","subtype":"Figure"},"ticker":{"type":"BasicTicker","id":"b70339961782cb7f9397e14c24ebe8a4"}}}]}}},"debug":false},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->
