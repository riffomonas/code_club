---
layout: post
title: "Smoothing lamb prices"
author: "PD Schloss"
date: 2020-06-25 11:55
blurb: "Rolling averages are in the news. We'll use <code>lag</code> to smooth our prices data"
comments: false
---

In [recent Code Club episodes](2020-06-18-comparing-lamb-prices) we have been looking at how to plot lamb prices from the United Producers livestock auction that I have been recording in a Google workbook. The data are pretty noisy. One way of smoothing the data has been to use `geom_smooth`. By adjusting the value of the `span` argument we have been able to make the smoothed line more (small values) or less squiggly (large values). With the ongoing COVID-19 pandemic there has been a lot in the news of 7-day rolling averages. Rolling averages are also called moving averages. The [Ann Arbor News](https://c0eib122.caspio.com/dp/74321000d8cf077bf4c747f4a37a) has been plotting daily case counts along with a rolling average for the state of Michigan. I noticed that this representation of the data puts the rolling average at the 4th day of the 7 day series. I haven't been able to find the raw data that were used to build the plot in an easy to access format. Naturally, this got me to thinking about how to calculate a rolling average for my noisy sheep price data.

For today's episode of Code Club, we'll build off of the code and concepts that we have have been working on for the past few weeks. Don't worry if you missed those episodes. We'll start with the block of code that we used last week and see how we can add the `lag` function from the `dplyr` package to help us calculate the rolling average. Along the way, we'll use the `mutate` and `pivot_longer` functions again, but in a new context. As always, don't watch the video straight through without firing up RStudio and trying the code and exercises yourself! Please be sure to see the [setup instructions](/code_club/setup-instructions) before you get going.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/L1J3M9jEWvQ" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Prompt

Over the past few episodes we've discussed `read_sheet`, `separate`, and `pivot_longer`. Here is the code chunk that we developed to "tidy" the data in my Google workbook that contains my sheep price data for my local livestock auction. Be sure to take your time to go through this code chunk to review the different topics we've discussed.

```r
library(tidyverse)
library(googlesheets4)

auction_data <- read_sheet("https://docs.google.com/spreadsheets/d/1_quMjJRBHDLQSmWQouzzyi1DOejAtCZnAeesdVyRWiQ/edit#gid=1467293328",
			sheet="numbers_and_prices",
			range="A:J",
			col_type = "Ddcccccccc",
			na="NA") %>%
		rename_all(tolower) %>%
		rename("aged_sheep" = "aged sheep",
			"feeder_lambs" = "feeder lambs",
			"hair_lambs" = "hair lambs",
			"new_crop" = "new crop",
			"small" = "40-85",
			"medium" = "85-105",
			"large" = "106-130",
			"extra_large" = ">131")


tidy_auction_data <- auction_data %>%
	pivot_longer(cols=-c("date", "total"), names_to="classes", values_to = "price_range") %>%
	separate(price_range, into=c("min", "max"), sep="-", convert=TRUE) %>%
	mutate(midpoint = (min+max)/2)
```


### Rolling averages

A rolling average is the average value of a set of observations that are typically collected over a period of time. They're used because they help to smooth noisy data. We've been seeing them in the reporting of COVID-19 data, but they're also commonly used to smooth noisy stock market data. A rolling average can be a lagging average where we average across observations before a point of interest, they can be leading where we average across observations before a point of interest, and they can also be centered where we average across observations where the point of interest is in the middle of the observations. We're interested in the lagging average.

To calculate a rolling average we need the value of the observation on the day of interest and the values that preceded it. Then we need to average across those values. To generate the values, we are going to use the `mutate` function to add the preceding days' values as separate columns to a data frame.

### `lag`

We can find values for the the preceding days' observations using the `lag` function. When you run `library(tidyverse)` you'll notice a warning that says, `dplyr::lag() masks stats::lag()`. This means that the packages `dplyr` and `stats` both have `lag` functions. If you use `?stats::lag` and `?dplyr::lag` you can see the differences. The upshot is that the `dplyr` version is easier to use and implement for our purposes.

As a simple example, consider the numbers 1 through 10

```r
x <- 1:10

lag(x) # the default lag is 1
lag(x, n=1)
lag(x, n=2)
lag(x, n=3)
```

You'll notice that the output vector is the same length as the input, but that the first `n` values are bumped to the right and replaced with `NA` values. If we have a series of dates and use `n=2`, then we are looking at the values starting from two days ago. One other argument to notice is `order_by`, which allows us to make sure that our values are in the correct order. For instance, we could chose to lag the `midpoint` column of our data frame by a week and we would want to order the observations by the `date` column. We'll see how to do this next


### Adding lagged columns to `tidy_auction_data`

With this in mind, we can create new columns for this week that we are interested in as well as the three previous weeks using `lag` and `mutate`. To keep things simple, let's only focus on the large class of lambs. To create a column with no lag (the observed value for the week) and column with the data for the previous week we can do this

```R
tidy_auction_data %>%
	filter(classes=="large") %>%
	mutate(lag0 = midpoint,
		lag1 = lag(midpoint, 1, order_by=date))
```

Since our data are weekly, let's expand this to create columns to calculate a 4 week rolling average.

```R
tidy_auction_data %>%
	filter(classes=="large") %>%
	mutate(lag0 = midpoint,
		lag1 = lag(midpoint, 1, order_by=date),
		lag2 = lag(midpoint, 2, order_by=date),
		lag3 = lag(midpoint, 3, order_by=date))
```

This gives us the data that we need to calculate our rolling average. We need to average across our four columns. My instinct is to do the following...

```R
tidy_auction_data %>%
	filter(classes=="large") %>%
	mutate(lag0 = midpoint,
		lag1 = lag(midpoint, 1, order_by=date),
		lag2 = lag(midpoint, 2, order_by=date),
		lag3 = lag(midpoint, 3, order_by=date)) %>%
	mutate(rolling_average = mean(c(lag0, lag1, lag2, lag3)))
```

This gives us a column of `NA` values because it is taking all of the rows across the four columns, calculating the mean, and then returning a single value that it repeats across all rows. We get the `NA` rather than a number because we're using the default `na.rm=FALSE` in `mean`. We want the mean for each row. You could get this to work if you used `group_by` and `summarize`

```R
tidy_auction_data %>%
	filter(classes=="large") %>%
	mutate(lag0 = midpoint,
		lag1 = lag(midpoint, 1, order_by=date),
		lag2 = lag(midpoint, 2, order_by=date),
		lag3 = lag(midpoint, 3, order_by=date)) %>%
	group_by(date) %>%
	summarize(midpoint = first(midpoint), rolling_average = mean(c(lag0, lag1, lag2, lag3)))
```

With that approach, I also needed to use the `first` function to get the observed `midpoint` value for each `rolling_average` value. The following is a bit simpler. It works because it returns a vector the same length as the original vectors that is used to build a new column.

```R
large <- tidy_auction_data %>%
	filter(classes=="large") %>%
	mutate(lag0 = midpoint,
		lag1 = lag(midpoint, 1, order_by=date),
		lag2 = lag(midpoint, 2, order_by=date),
		lag3 = lag(midpoint, 3, order_by=date)) %>%
	mutate(rolling_average = (lag0 + lag1 + lag2 + lag3)/4) %>%
	select(date, midpoint, rolling_average)
```

That works pretty nicely and gives the same values as we saw using the `group_by` and `summarize` approach. I went ahead and got rid of all of the columns except for `date`, `midpoint`, and `rolling_average` to make it easier to see the data we're interested in and saved it as the variable `large`.


### Plotting the observed and averaged data

Let's see what our data look like with and without the 4-week rolling average. We currently have `midpoint` and `rolling_average` as separate columns. To plot the data together we need to tidy our data frame to get these values together.

```R
large %>%
	pivot_longer(-date, names_to="method", values_to="price")
```

Now we're ready to add on our `ggplot` functions to plot `date` on the x-axis, `price` on the y-axis, and `method` as the color

```R
large %>%
	pivot_longer(-date, names_to="method", values_to="price") %>%
	ggplot(aes(x=date, y=price, color=method)) +
		geom_line()
```

Great! I'm going to clean up the plot a bit as we have done in previous weeks

```R
large %>%
	pivot_longer(-date, names_to="method", values_to="price") %>%
	ggplot(aes(x=date, y=price, color=method)) +
		geom_line() +
		theme_light() +
		labs(x="Date",
			y="Price ($/100 lbs)",
			title="A rolling average smooths the noisiness of the large lamb prices",
			subtitle="Lagging four week rolling average of the midpoint prices",
			caption="Prices as reported from United Producers in Manchester, MI")
```

The colors are a bit trippy. I'd like to change the colors of the lines so that the `midpoint` data are `gray` and the `rolling_average` data are `dodgerblue`. We can do this with `scale_color_manual`


```R
large %>%
	pivot_longer(-date, names_to="method", values_to="price") %>%
	ggplot(aes(x=date, y=price, color=method)) +
		geom_line() +
		theme_light() +
		labs(x="Date",
			y="Price ($/100 lbs)",
			title="A rolling average smooths the noisiness of the large lamb prices",
			subtitle="Lagging four week rolling average of the midpoint prices",
			caption="Prices as reported from United Producers in Manchester, MI") +
		scale_color_manual(values=c("gray", "dodgerblue"))
```

That makes it nicer to look at. We can compare this last plot to

```r
large %>%
	ggplot(aes(x=date, y=midpoint)) +
		geom_line(color="gray") +
		geom_smooth(size=0.5, span=0.1, se=FALSE)
```

I kind of prefer the `geom_smooth` approach since it does a better job of handling the missing data that seemed to occur frequently in 2019. Which do you prefer?


## Exercises

1\. How would you modify the code we generated to calculate a 3 week rolling average? 5 week?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">

```R
large_three <- tidy_auction_data %>%
	filter(classes=="large") %>%
	mutate(lag0 = midpoint,
				 lag1 = lag(midpoint, 1, order_by=date),
				 lag2 = lag(midpoint, 2, order_by=date)) %>%
	mutate(rolling_average = (lag0 + lag1 + lag2)/3) %>%
	select(date, midpoint, rolling_average)
```

or

```R
large_five <- tidy_auction_data %>%
	filter(classes=="large") %>%
	mutate(lag0 = midpoint,
				 lag1 = lag(midpoint, 1, order_by=date),
				 lag2 = lag(midpoint, 2, order_by=date),
				 lag3 = lag(midpoint, 3, order_by=date),
				 lag4 = lag(midpoint, 4, order_by=date)) %>%
	mutate(rolling_average = (lag0 + lag1 + lag2 + lag3 + lag4)/5) %>%
	select(date, midpoint, rolling_average)
```

</div>


2\. We covered calculating the rolling average for the large class of lambs. How would we calculate the rolling averages for each class of lambs? Build a plot showing the prices for the small, medium, large, and extra large classes of lambs.

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">

```r
rolling <- tidy_auction_data %>%
	group_by(classes) %>%
		mutate(lag0 = midpoint,
				 lag1 = lag(midpoint, 1, order_by=date),
				 lag2 = lag(midpoint, 2, order_by=date),
				 lag3 = lag(midpoint, 3, order_by=date)) %>%
	mutate(rolling_average = (lag0 + lag1 + lag2 + lag3)/4) %>%
	select(date, classes, midpoint, rolling_average)

rolling %>%
	filter(classes=="small" | classes=="medium" | classes=="large" | classes=="extra_large") %>%
	ggplot(aes(x=date, y=rolling_average, color=classes)) + geom_line()
```
</div>


3\. We've seen how we can repackage other functions we've already learned with the `lag` function to generate a rolling average. As we saw in the first exercise, pivoting between different lag lengths can be a bit tedious. Also, if we wanted to do something like an average over 50 values, that would be horrible. Thankfully, there's a package called `zoo` that has a function, `rollmean`. Install `zoo` and look at the `?rollmean` documentation and see whether you can figure out how to replace our `mutate` functions with a single `mutate` function that calls `rollmean`

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">

```r
library(zoo)

tidy_auction_data %>%
	filter(classes=="large") %>%
	mutate(rolling_average = rollmean(midpoint, 4, na.pad=TRUE, align="right")) %>%
	select(date, midpoint, rolling_average)
```
</div>
