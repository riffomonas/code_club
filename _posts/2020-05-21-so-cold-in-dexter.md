---
layout: post
title: "So cold in Dexter"
author: "PD Schloss"
date: 2020-05-21 12:00
blurb: "My wife planted a garden, was it too soon? We'll use <code>mutate</code>, <code>filter</code>, <code>group_by</code>, and <code>summarize</code> to find out"
comments: false
---

A couple of weeks ago, my wife conned our 15 year old into roto tilling up a part of our pig pen so she could have a garden. She's slowly starting to plant stuff in it. We also have about 20 apple trees that have started to blossom. I *think* we're past the point of worrying about hard frosts that could kill germinating plants or apple blossoms. But how sure should I be? I thought this was a great question for a code club. If you google the name of your town and "frost dates" you'll find a bunch of gardening websites that give guidance on when to plant in the spring and when to expect your harvesting to be done in the fall. Of course we could find one of those websites, but where's the fun in that?

Today's code club will answer this and other questions using functions we've already seen including `filter`, `group_by`, and `summarize`. We'll focus on how we can use the `mutate` function to create new columns in our data frames. To spice things up a bit we'll also talk about how we can use arithmetic functions with logical variables. By the end of today's Code Club we'll know when I can stop worrying about my wife's garden and my apple trees.

Don't watch the video straight through without firing up RStudio and trying the code and exercises yourself! Please be sure to see the [setup instructions](/code_club/setup-instructions) before you get going.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/LUnV44uFr2c" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Prompt
Like we did last week, I used the NOAA website to track down [daily high and low temperatures](https://www.ncdc.noaa.gov/cdo-web/datasets/GHCND/locations/CITY:US260002/detail) going back to 1891 for Ann Arbor, MI. You can find similar data for your favorite place by using the [Climate Data Online Search](https://www.ncdc.noaa.gov/cdo-web/search) to find "Daily Summaries" datasets. Be sure to click on the specific station you want at the bottom of your location's page that has the longest history.

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

### Breaking down our approach

My question boils down to what is the probability of having a temperature below freezing on each day of the year. Alternatively, we could ask, across the last 130 years, how many times has there been a low temperature below freezing on each day of the year. To do this, we need to do a few things...

1. Determine whether a low temperature was below freezing for every day in our dataset
1. Aggregate our data by month and day
1. Determine the fraction of month-day combinations had a low temperature below freezing across all of the years in our dataset
1. Find the day in May where there's a below 5 or 10% risk of having another day with below freezing temperatures.

If you've been following along with past Code Clubs, you already know how to do most of these steps. Now to convert these steps to R code!


### The `mutate` function

That code block has a line that tells R to create three new columns - `year`, `month`, and `day`. That function is `mutate`. We can use `mutate` to write over existing columns or to create new columns. In this example, I used `mutate` along with the `year`, `month`, and `day` functions from the `lubridate` package to extract information from the `date` column of our data frame. Let's step back and see how we can use `mutate` to add a column.

The `mutate` function takes as its arguments the name of the new column (e.g. `year`), an equal sign (i.e. `=`), and a formula telling R how to calculate the values for the new column. We want the output of the formula to be a single value. The `?mutate` page calls these "Name-value pairs". We can have multiple name-value pairs that we set apart with commas as I did in the code chunk above. There are other `mutate` functions including `mutate_if`, `mutate_at`, `mutate_all`, `transmutate`, `transmutate_if`, `transmutate_at`, and `transmutate_all`. You can learn more about them at their help pages, but honestly, I don't think I've ever used them.

To determine whether a given day had a low temperature below freezing, we will use the mutate function to create a new colum, `below_freezing`, that will have a logical value of `TRUE`, `FALSE`, or `NA` if the low temperature was not reported that day. The low temperature is recorded in the `TMIN` column so asking `TMIN < 0` will tell us whether the tempreature was below freezing.

```R
aa_weather %>%
	mutate(below_freezing = TMIN < 0)
```

### Arithmetic with logicals

Having `below_freezing` be a logical allows us to do some slick things. We can treat logicals as numbers and use them in functions like `sum` and `mean`. The key is to remember that `TRUE` is `1` and `FALSE` is `0`. Probably the best way to remember this is that zero and `FALSE` are both nothing. If you can forget, you can always use `as.numeric`...

```R
my_vector <- c(TRUE, FALSE, TRUE, TRUE, FALSE)

as.numeric(my_vector)
```

If we use `my_vector` with an arithmetic function, it will convert the logical value to a zero or one for us before evaluating the function.

```R
sum(my_vector)
mean(my_vector)
```

The `sum` function tells us that there are 3 `TRUE` values and the `mean` function tells us that 0.6 of the values were `TRUE`. Although `mean(my_vector)` is more concise, we could have also done

```R
sum(my_vector)/length(my_vector)
```

### Getting a fraction with `summarize`

Do you see where we're going with this? We can use `mean(below_freezing)` in `summarize` to get the fraction of years a day was below freezing.

```R
aa_weather %>%
	mutate(below_freezing = TMIN < 0) %>%
	summarize(frac_below_freezing = mean(below_freezing))
```

That gives an `NA` value because some of the days did not have a `TMIN` value. To ignore those values, we can use the `na.rm=TRUE` argument in `mean`

```R
aa_weather %>%
	mutate(below_freezing = TMIN < 0) %>%
	summarize(frac_below_freezing = mean(below_freezing, na.rm=TRUE))
```

### Getting our daily fraction using `group_by` and `summarize`

Cool, 35% of the days since 1891 have had a low temperature below freezing. But we wanted to aggregate the month-day combinations and then get the fraction across all  years. We've seen this before with the `group_by`/`summarize` combination

```R
aa_weather %>%
	mutate(below_freezing = TMIN < 0) %>%
	group_by(month, day) %>%
	summarize(frac_below_freezing = mean(below_freezing, na.rm=TRUE))
```


### Using `filter` to focus in on May

Let's focus in on the dates in May using the `filter` function

```R
aa_weather %>%
	mutate(below_freezing = TMIN < 0) %>%
	group_by(month, day) %>%
	summarize(frac_below_freezing = mean(below_freezing, na.rm=TRUE)) %>%
	filter(month == 5) %>%
	print(n=31)
```

Based on historic weather data, the garden and apple trees are probably safe after the 10th of May.

```R
aa_weather %>%
	mutate(below_freezing = TMIN < 0) %>%
	group_by(month, day) %>%
	summarize(frac_below_freezing = mean(below_freezing, na.rm=TRUE)) %>%
	filter(month == 5) %>%
	print(n=31)
```

It has only been below freezing 11 times after the 10th in the past 130 years (8.5%). If we wanted to be more conservative, we could wait until the 13th since there have only been freezing temperatures on or after the 13th of May, 6 times (4.6%). Sure enough, if we look at the [Gardening in the Mitten](https://www.gardeninginthemitten.com/michigan-frost-dates/) website, they tell us that Ann Arbor's spring frost should end by May 10th and the first fall frost will be October 5th

Hopefully, you feel more comfortable using `mutate`, `filter`, `group_by`, and `summarize` now that you've seen them used in a new context. Are you ready to answer some questions using these strengthened skills?


## Assignment

1\. When do you think the vegetables in our garden will be done growing?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```r
aa_weather %>%
	mutate(below_freezing = TMIN < 0) %>%
	group_by(month, day) %>%
	summarize(frac_below_freezing = mean(below_freezing, na.rm=TRUE)) %>%
	filter(month == 10) %>%
	print(n=30)
```

If you had any growth after the 18th of the month, that would be a surprise (90% of the years have had a frost on the 18th or before). This is about two weeks longer than the Gardening in the Mitten suggested. I suspect they used the 5th because 10% of the years had a frost on that day or earlier.
</div>


2\. Add a column to `aa_weather` that represents the high temperature for the day in Fahrenheit. Using this new column, determine the average number of days per year where the temperature is over 90 F.

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```r
aa_weather %>%
	mutate(t_high_f = 9/5 * TMAX + 32,
		hot = t_high_f > 90) %>%
	group_by(year) %>%
	summarize(total_hot_days = sum(hot, na.rm=T)) %>%
	filter(year > 1891 & year < 2020) %>%
	summarize(ave_hot_days = mean(total_hot_days))
```
</div>


3\. I've heard a rule of thumb that if a day's low and high temperature is over 100 F, then grass will grow. When can I expect the grass to start growing reliably in May?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```r
aa_weather %>%
	mutate(t_high_f = 9/5 * TMAX + 32,
		t_low_f = 9/5 * TMIN + 32,
		grass_growing = t_high_f + t_low_f > 100) %>%
	filter(year > 1891 & year < 2020) %>%
	group_by(month, day) %>%
	summarize(frac_growing_days = mean(grass_growing, na.rm=TRUE)) %>%
	filter(month == 5) %>%
	print(n=31)
```

If this rule of thumb is true, then we won't have consistent temperatures that are warm enough for grass growth until the 23rd of May.
</div>


---

Title credit: The Cranberries, [So cold in Ireland](https://www.youtube.com/watch?v=WX2TXMJXS4o)
