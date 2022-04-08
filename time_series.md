# Time series

Time series, by definition, is a sequence of data point collected over a certain period of time. In this chapter, we will demonstrate several useful ways of plotting time-series data and how to processing ``date`` data type in R.


## Dates

Since time series analysis looks into how data is changing over time, the very first step is to transform the data into correct format.

### Basic R functions

You can convert character data to ``Date`` class with ``as.Date()``:


```r
dchar <- "2018-10-12"
ddate <- as.Date(dchar)
class(dchar)
```

```
## [1] "character"
```

```r
class(ddate)
```

```
## [1] "Date"
```
You can also specifying the format by:


```r
as.Date("Thursday, January 6, 2005", format = "%A, %B %d, %Y")
```

```
## [1] "2005-01-06"
```
Here is a list of the conversion specifications for date format from [this post](https://michaeltoth.me/the-ultimate-opinionated-guide-to-base-r-date-format-functions.html)

<center>
![](images/common_r_date_formats.png){width=75%}
</center>

Also, ``Date`` class supports calculation between dates:


```r
as.Date("2017-11-02") - as.Date("2017-01-01")
```

```
## Time difference of 305 days
```

```r
as.Date("2017-11-12") > as.Date("2017-3-3")
```

```
## [1] TRUE
```
### Lubridate

The tidyverse ``lubridate`` makes it easy to convert dates that are not in standard format with ``ymd()``, ``ydm()``, ``mdy()``, ``myd()``, ``dmy()``, and ``dym()`` (among many other useful date-time functions):


```r
lubridate::mdy("April 13, 1907")
```

```
## [1] "1907-04-13"
```

The ``lubridate`` package also provides additional functions to extract information from a date:


```r
today <- Sys.Date()
lubridate::year(today)
```

```
## [1] 2022
```

```r
lubridate::yday(today)
```

```
## [1] 95
```

```r
lubridate::month(today, label = TRUE)
```

```
## [1] Apr
## 12 Levels: Jan < Feb < Mar < Apr < May < Jun < Jul < Aug < Sep < ... < Dec
```

```r
lubridate::week(today)
```

```
## [1] 14
```

## Time series

For time-series data-sets, line plots are mostly used with time on the x-axis. Both base R graphics and ``ggplot2`` “know” how to work with a Date class variable, and label the axes properly:

The data comes from the official [website](https://www.freddiemac.com/pmms).


```r
library(dplyr)
library(readxl)
library(tidyr)
library(ggplot2)
df <- read_excel("data/historicalweeklydata.xls", 
    col_types = c("date", "numeric", "numeric", 
        "numeric"))

plot(df$Week, df$`30 yr FRM`, type = "l") # on the order of years
```

<img src="time_series_files/figure-html/unnamed-chunk-6-1.png" width="384" style="display: block; margin: auto;" />

```r
g<-ggplot(df %>% filter(Week < as.Date("2006-01-01")), 
       aes(Week, `30 yr FRM`)) + 
  geom_line() + 
  theme_grey(14)
g
```

<img src="time_series_files/figure-html/unnamed-chunk-6-2.png" width="384" style="display: block; margin: auto;" />

We can control the x-axis breaks, limits, and labels with ``scale_x_date()``, and use ``geom_vline()`` with ``annotate()`` to mark specific events in a time series.


## Multiple time series

The following plot shows a multiple time series of U.S. Mortgage rates. 


```r
df2 <- df %>% pivot_longer(cols = -c("Week"), names_to = "TYPE") %>%
  mutate(TYPE = forcats::fct_reorder2(TYPE, Week, value))# puts legend in correct order

ggplot(df2, aes(Week, value, color = TYPE)) +
  geom_line() +
  ggtitle("U.S. Mortgage Rates") +  labs (x = "", y = "percent") +
  theme_grey(16) +
  theme(legend.title = element_blank())
```

<img src="time_series_files/figure-html/unnamed-chunk-7-1.png" width="576" style="display: block; margin: auto;" />

To plot the time series in a specific period of time, use ``filter()`` before ``ggplot``:


```r
library(lubridate)
df2010 <- df2 %>% filter(year(Week) == 2010)
ggplot(df2010, aes(Week, value, color = TYPE)) +
  geom_line() +
  ggtitle("U.S. Mortgage Rates")
```

<img src="time_series_files/figure-html/unnamed-chunk-8-1.png" width="576" style="display: block; margin: auto;" />