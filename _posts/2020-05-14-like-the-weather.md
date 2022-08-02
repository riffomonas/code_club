---
layout: post
title: "Like the weather"
author: "PD Schloss"
date: 2020-05-14 12:00
blurb: "Gee whiz it was cold Saturday morning. Is this normal? We'll use <code>filter</code>, <code>group_by</code>, and <code>summarize</code> to find out"
comments: false
---

Last Saturday (May 9th) was pretty cold here in southeastern Michigan. It was 26F (-3C) when I woke up. Although we've had some warm days, this Spring has felt pretty cold. Is that normal? If only we had the skills to answer such a question... In fact, we do! Today's code club will answer this and other questions using functions we've already seen including `filter`, `group_by`, and `summarize`. By the end of today's Code Club we'll know how normal Friday's temperatures were and we'll have a greater familiarity with using these functions. In addition, we'll learn how to use the `arrange` and `top_n` functions to find out which previous year had the lowest temperature.

Don't watch the video straight through without firing up RStudio and trying the code and exercises yourself! Please be sure to see the [setup instructions](/code_club/setup-instructions) before you get going.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/tVKcVKORmow" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Prompt
I used the NOAA website to track down [daily high and low temperatures](https://www.ncdc.noaa.gov/cdo-web/datasets/GHCND/locations/CITY:US260002/detail) going back to 1891 for Ann Arbor, MI. You can find similar data for your favorite place by using the [Climate Data Online Search](https://www.ncdc.noaa.gov/cdo-web/search) to find "Daily Summaries" datasets. Be sure to click on the specific station you want at the bottom of your location's page that has the longest history.

I have Ann Arbor's data stored on GitHub and will use `read_csv` to read it in and do some small touch ups to read the data in and extract the year, month, and day from the date. We'll get some warning messages, which we can ignore for this Code Club.

```R
library(tidyverse)
library(lubridate)

aa_weather <- read_csv("https://github.com/riffomonas/generalR_data/blob/master/noaa/USC00200230.csv?raw=true",
	col_type=cols(TOBS = col_double())) %>%
	select(DATE, TMAX, TMIN, TOBS) %>%
	mutate(year = year(DATE),
		month=month(DATE),
		day = day(DATE))
```


### Using the `filter` and `summarize` functions to find normal for one day

My first question is how weird were Friday's temperatures. Recall that we can use the `filter` function to retrieve rows from a data frame that match a set of specifications. Here we want the rows that correspond to the 5th month (i.e. May) and the 9th day.


```r
may_nineth <- aa_weather %>%
	filter(month == 5 & day == 9)
```

We'll see that `may_nineth` is a dataframe with 127 rows and 7 columns including each day's high (TMAX), low (TMIN), and observed (TOBS) temperature for the day. It's hard to look through `may_nineth` to see how typical our temperatures were. Instead, we can use the `summarize` function. Previously, we used summarize with the `group_by` function. We can also use it on non-grouped data. Let's find the average high, low, and observed temperatures. Remember to use `na.rm=T` to ignore those temperatures where the value is `NA`

```r
may_nineth %>%
	summarize(ave_high = mean(TMAX, na.rm=T),
		ave_low = mean(TMIN, na.rm=T),
		ave_obs = mean(TOBS, na.rm=T),
		N = n())
```

That shows us the average temperatures and the number of years of data we have. We can use `quantile` to define a range of values. To get the 95% confidence interval, we need to set the probability for the lower bound to 0.025 (i.e. 2.5%) and the upper bound to 0.975 (i.e. 97.5%) - the range is 95%. For the high temperature...

```r
may_nineth %>%
	summarize(ave_high = mean(TMAX, na.rm=T),
		lci_high = quantile(TMAX, prob=0.025, na.rm=T),
		uci_high = quantile(TMAX, prob=0.975, na.rm=T),
		N = n())
```

And to get the same information for the lows we can do...

```r
may_nineth %>%
	summarize(ave_high = mean(TMAX, na.rm=T),
		lci_high = quantile(TMAX, prob=0.025, na.rm=T),
		uci_high = quantile(TMAX, prob=0.975, na.rm=T),
		ave_low = mean(TMIN, na.rm=T),
		lci_low = quantile(TMIN, prob=0.025, na.rm=T),
		uci_low = quantile(TMIN, prob=0.975, na.rm=T),
		N = n())
```

We get one row back because we already filtered the data to get each year's data for May 9th. We can see that Friday's temperatures were lower than we'd expect by the 95% confidence interval.


### Finding the historic highs and lows with `arrange` and `top_n`

What was the lowest temperature recorded on May 9th? As with anything in R, there are many ways to do this. We could use summarize

```r
may_nineth %>%
	summarize(min_low = min(TMIN))

may_nineth %>%
	summarize(max_high = max(TMAX))
```

Using summarize with `min` or `max` returns a single number but doesn't tell us the year. We could also use `arrange` to sort `may_nineth` by `TMIN` or `TMAX`

```r
may_nineth %>%
	arrange(TMIN)
```

The previous low was -2.2C in 1947. If we want a descending sort, we need to use the `desc` function

```r
may_nineth %>%
	arrange(desc(TMAX))
```

The historic high for the day is 30 and was achieved in 1926, 1965, 2014, and 2015. A third approach we'll discuss is the `top_n` function. We can add `top_n` with the column name we want to pull the top rows by along with the number of rows we want. We can also give the function a negative number of rows to get the smallest values

```r
may_nineth %>%
	top_n(TMAX, n=3)

may_nineth %>%
	top_n(TMIN, n=-3)
```

We see that the output isn't sorted by `TMAX` or `TMIN` and that it may return more rows than we asked for if there are ties.


### Using `group_by` to get the highs and lows for any day of the year
Aside from being cold this year, there's nothing special about May 9th. We might want similar information for any day of the year. If we put our pipeline together this was what we had to get the ranges for May 9th over the past 127 years

```r
aa_weather %>%
	filter(month == 5 & day == 9) %>%
	summarize(ave_high = mean(TMAX, na.rm=T),
		lci_high = quantile(TMAX, prob=0.025, na.rm=T),
		uci_high = quantile(TMAX, prob=0.975, na.rm=T),
		ave_low = mean(TMIN, na.rm=T),
		lci_low = quantile(TMIN, prob=0.025, na.rm=T),
		uci_low = quantile(TMIN, prob=0.975, na.rm=T),
		N = n())
```

If we wanted to change this to get the same data for any day of the year, we can replace our `filter` function with a `group_by` on the `month` and `day` columns

```r
daily_t_summary <- aa_weather %>%
	group_by(month, day) %>%
	summarize(ave_high = mean(TMAX, na.rm=T),
		lci_high = quantile(TMAX, prob=0.025, na.rm=T),
		uci_high = quantile(TMAX, prob=0.975, na.rm=T),
		ave_low = mean(TMIN, na.rm=T),
		lci_low = quantile(TMIN, prob=0.025, na.rm=T),
		uci_low = quantile(TMIN, prob=0.975, na.rm=T),
		N = n())
```

Checking that we get what we had before for May 9th, we can add a `filter` function call.

```r
daily_t_summary %>%
	filter(month == 5, day == 9)
```

Here are the temperatures for my birthday...

```r
daily_t_summary %>%
	filter(month == 6, day == 20)
```

Hopefully, you feel more comfortable using `filter`, `group_by`, and `summarize` now that you've seen them used in a new context. Are you ready to answer some questions using these strengthened skills?


## Assignment

1\. Which year was the hottest on your birthday? Which was the hottest since you were born?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```r
aa_weather %>%
	filter(month == 6 & day == 20) %>%
	top_n(TMAX, n=1)
```

I was born in 1976...

```r
aa_weather %>%
	filter(month == 6 & day == 20 & year >= 1976) %>%
	top_n(TMAX, n=1)
```
</div>


2\. What were the hottest and coldest temperatures recorded the year you were born?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```r
aa_weather %>%
	filter(year == 1976) %>%
	summarize(coldest = min(TMIN),
		hottest = max(TMAX))
```
</div>


3\. Calculate the average high temperature for each year between 1892 and 2019 (the years we have complete data for). What was the average temperature the year you were born? Which are the coldest and hottest years we have data for (based on the annual average high temperature)?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```r
annual_tmax <- aa_weather %>%
	filter(year >= 1892 & year <= 2019) %>%
	group_by(year) %>%
	summarize(ave_tmax = mean(TMAX, na.rm=T),
		lci_tmax = quantile(TMAX, prob=0.025, na.rm=T),
		uci_tmax = quantile(TMAX, prob=0.975, na.rm=T),
		n = n())

annual_tmax %>% filter(year == 1976)

annual_tmax %>% top_n(ave_tmax, n=1)

annual_tmax %>% top_n(ave_tmax, n=-1)
```
</div>


4\. What is the average high temperature and 95% confidence interval for each month of the year? What is it for your birth year?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```r
month_tmax <- aa_weather %>%
	group_by(month) %>%
	summarize(ave_tmax = mean(TMAX, na.rm=T),
		lci_tmax = quantile(TMAX, prob=0.025, na.rm=T),
		uci_tmax = quantile(TMAX, prob=0.975, na.rm=T),
		n = n())

month_tmax %>% filter(month == 6)
```
</div>

5\. Get the weather data for your favorite place using the approach Pat outlines in the video. What will the temperature be there on your birthday?


---

Title credit: 10,000 Maniacs, [Like the Weather](https://www.youtube.com/watch?v=te7bbWBXusk)
