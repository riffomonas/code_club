---
layout: post
title: "Predicting lamb futures"
author: "PD Schloss"
date: 2020-06-11 11:55
blurb: "How much do lamb prices fluctuate at the local livestock auction? We'll track the variation with <code>separate</code>, <code>geom_smooth</code>, and <code>geom_ribbon</code>"
comments: false
---

As I've mentioned over the past few code clubs, I have sheep on my farm. I like raising sheep because they have a pretty quick growing period of about six months, they're small enough that I can have a decent number of them, they taste good, and in southeastern Michigan there's a pretty strong Middle Eastern clientele that eat a lot of lamb. Last week, we created a figure showing the number of lambs being sold at my local livestock auction. That's only part of the equation - lamb numbers could be dropping because prices are falling or with the decline in sheep, they could be getting more valuable. Also, you may have noticed this, but the total number of sheep sold in the figure from last week was the total - I don't know how many animals were sold in each weight category. Although I've been recording the number of sheep sold and the price for each class for the past 5 years, I've never really plotted the data. Today we'll plot the price for each weight class to see what has happened to the prices over the past 5 years. As a reminder, we saw that sheep sales began to fall about 2 or 3 years ago.

For today's Code Club, we'll build off of what we did last week using the `googlesheets4` package to read that spreadsheet, we'll remember how to use the `rename` function to fix my bad column names, we'll learn to use the `separate` function to split my price ranges into two columns, and we'll see how we can use the `geom_ribbon` function from the `ggplot2` package to plot the data. Along the way we'll also see the `mutate`, `geom_line`, and `geom_smooth` functions again. Finally, we'll use `ggplot2`'s `labs` function to give our plots descriptive titles and axis labels. Don't watch the video straight through without firing up RStudio and trying the code and exercises yourself! Please be sure to see the [setup instructions](/code_club/setup-instructions) before you get going.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/I5azya4jHoc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Prompt

Today we'll use the `tidyverse` package, but we'll also use the [`googlesheets4`](https://googlesheets4.tidyverse.org) package for reading my googlesheet. If you haven't already, you'll need to instal it as [described last week](2020-06-04-eat-more-lamb). We'll read in the spreadsheet and go ahead and clean up the column names. Things like `Aged sheep` or `>131` don't work very well with R. Of course, I could change my spread sheet, but I'd like to use `rename` to practice [those skills](2020-04-02-the-data-are) and in case anyone else is using the spreadsheet I don't want to break their code.



```R
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
```

### Separating columns

For today's task, I'd like to take the large lambs (those weighing between 105 and 130 pounds) and see how the high and low prices have varied over the course of the time I've been tracking these data. In the spreadsheet I have a price stored as `150-170`. This is the range in prices from the low to high price per 100 lbs of lamb. In other words, for that week, the prices ranged from $1.50 to $1.70 per live pound of lamb. Since each row has the same `<min>-<max>` format, it's relatively straightforward to separate the data into two columns using `dplyr`'s `separate` function.

The syntax for `separate` requires that you give it a column that you want to separate and the column names that you want for the new columns. The `separate` function will guess what character you want to separate on, but it's safest and more descriptive to provide a value to the `sep` argument.

```r
auction_data %>%
	separate(large, sep="-", into=c("min", "max")) #try running this without the sep argument
```

You'll notice a couple of things. First, we no longer have the `large` column. We could use `remove=FALSE` to keep that column if we wanted to keep it around. Second, my `min` and `max` columns are character types although they are actually numbers now. We can get `separate` to convert these columns to numbers using `convert=TRUE`. Let's include these arguments along with a `select` function call to make it easier to see what we've done

```r
auction_data %>%
	separate(large, sep="-", into=c("min", "max"), remove=FALSE, convert=TRUE) %>%
	select(date, large, min, max)
```

Nice, eh? I don't need the `large` column, so let's get rid of that from our output and save the output as `large_prices`

```r
large_prices <- auction_data %>%
	separate(large, sep="-", into=c("min", "max"), convert=TRUE) %>%
	select(date, min, max)
```


### Plotting the average price

We don't have more fine scale price data for the weight class to know whether larger lambs had a lower price or what the average price was. The reality is that the true prices are kind of chunky. At the auction they may sell a dozen lambs together for the same price and other times they may sell one or two lambs per lot. Also, just because two lambs are 120 pounds, doesn't mean that their carcass qualities are going to be the same - so weight isn't everything. The range is really the best statistic to report. Let's create a line plot of the midpoint between the max and min price using `mutate` and `geom_line`. We'll start with creating the midpoint

```r
large_prices %>%
	mutate(mid_point = (min + max) / 2)
```

And we can plot these data like we did last week with `geom_line`

```r
large_prices %>%
	mutate(mid_point = (min + max) / 2) %>%
	ggplot(aes(x=date, y=mid_point)) +
		geom_line()
```

Wow, that's pretty seasonal! Let's fit a smoothed line to the data using the `span` argument in `geom_smooth`

```r
large_prices %>%
	mutate(mid_point = (min + max) / 2) %>%
	ggplot(aes(x=date, y=mid_point)) +
		geom_line() +
		geom_smooth(span=0.1)
```

This shows that the prices have fallen with the number of lambs sold. Interestingly, they seem to spike in April and May, but this year the spike fell off in March when the pandemic hit.


### Plotting the range of prices

Again, the midpoint of the min and max price, really isn't the mean or median price. We can use `geom_ribbon` to plot the range. As the name suggests, it will plot the data to look like a ribbon. For this geom, we need to give it columns corresponding to `ymin` and `ymax`

```r
large_prices %>%
	ggplot(aes(x=date, ymin=min, ymax=max)) +
	geom_ribbon()
```

We can also provide the `fill` argument to `geom_ribbon` to set the color of the ribbon. If you want to set the color of the boundary line on the ribbon, you would use the `color` argument. Normally, `color` is set to `NA`, which means it has no color.

```r
large_prices %>%
	ggplot(aes(x=date, ymin=min, ymax=max)) +
	geom_ribbon(fill="dodgerblue", color="black")
```

Unfortunately, there isn't an easy way to smooth the ribbon boundary line.


### Adding titles and fixing axis labels

I'm going to go ahead and add the `theme_light` to this plot to make it a little cleaner

```r
large_prices %>%
	ggplot(aes(x=date, ymin=min, ymax=max)) +
	geom_ribbon(fill="dodgerblue", color="black") +
	theme_light()
```

I'd also like to add some better labelling to the plot to make it clear what's going on. For example, the plot doesn't have a title and the y-axis isn't labelled. We can fix this with the `ggplot2` function, `labs`. The `labs` function will take arguments to set the main `title` and `subtitle`, a `caption`, and the `x` and `y` labels. To see where these will go, let's give the arguments their own name as their value

```r
large_prices %>%
	ggplot(aes(x=date, ymin=min, ymax=max)) +
	geom_ribbon(fill="dodgerblue", color="black") +
	labs(title="main", subtitle="sub", caption="caption", x="x", y="y") +
	theme_light()
```

Let's fill them in with more meaningful values

```r
large_prices %>%
	ggplot(aes(x=date, ymin=min, ymax=max)) +
	geom_ribbon(fill="dodgerblue", color="black") +
	labs(title="Large lambs have their highest value in April and May",
			subtitle="Prices for lambs between 106 and 130 pounds",
			caption="Data as reported by United Producers in Manchester, MI",
			x="Date",
			y="Price ($/100 lbs)") +
	theme_light()
```

I have to admit that although I've been recording these data for the past 5 years, I did not realize that prices for these large lambs was so consistently seasonal. This is valuable information! If a lamb's value peaks in April and May and it takes about 6 months to get a lamb to 110 pounds, then I likely want to have a group of lambs born in October or November. This plot also causes me to ask other questions - are other weight classes this seasonal? How much has the range in prices varied over this period? These are the types of questions we'll take on now in the exercises.


## Exercises

1\. "Aged sheep" are those sheep that are older than 1 year old. Typically, they're ewes that aren't able to support a lamb or rams that are no longer needed for breeding. These animals are generally used to produce pet food (it's unlikely that the "lamb" in fido's kibble is technically lamb). In Southeastern Michigan, there are anecdotes that some buyers like these animals because they are larger, cheaper, and have a flavor that they like. Is there any seasonal variation to the price in aged sheep? Produce a fitted line plot using the mid-point and a ribbon plot showing the range. Be sure to include an informative title and labels.


<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```r
aged_sheep <- auction_data %>%
	separate(aged_sheep, sep="-", into=c("min", "max"), convert=TRUE) %>%
	select(date, min, max) %>%
	mutate(midpoint = (min+max)/2)

aged_sheep %>%
	ggplot(aes(x=date, y=midpoint)) +
		geom_line(color="gray") +
		geom_smooth(span=0.1, se=FALSE) +
		labs(title="Aged sheep have their highest value at the beginning of the year",
				subtitle="Prices for aged sheep",
				caption="Data as reported by United Producers in Manchester, MI",
				x="Date",
				y="Price ($/100 lbs)") +
		theme_light()

aged_sheep %>%
	ggplot(aes(x=date, ymin=min, ymax=max)) +
	geom_ribbon(fill="dodgerblue", color="black") +
		labs(title="Aged sheep have their highest value at the beginning of the year",
				subtitle="Prices for aged sheep",
				caption="Data as reported by United Producers in Manchester, MI",
				x="Date",
				y="Price ($/100 lbs)") +
	theme_light()
```

Another surprise to me! Although it varies by year, aged sheep generally sell better at the beginning of the year. This is likely because farmers have already culled their aged sheep going into the winter, which is the time of year when its the most expensive to feed sheep.
</div>


2\. How has the weekly range in price for large lambs varied over the past 5 years?


<input type="button" class="hideshow">
<div markdown="1" style="display:none;">

```r
large_prices %>%
	mutate(range = max - min) %>%
	ggplot(aes(x=date, y=range)) +
		geom_line(color="gray") +
		geom_smooth(span=0.1, se=FALSE) +
			labs(title="Although there's been seasonal variation in large lamb prices, COVID-19 has\nbeen a major perturbation to the lamb market",
					subtitle="Range of prices for large lambs (106-130 lbs) each week",
					caption="Data as reported by United Producers in Manchester, MI",
					x="Date",
					y="Difference in high and low prices ($/100 lbs)") +
		theme_light()

```
</div>


3\. I can sell lambs at 95 pounds or at 125 pounds. If we assume that these lambs would get the midpoint price for their weight class, what is the total difference in price I would receive for each animal? How has this varied over the past 5 years. This difference would tell me the most it should cost me to have an animal gain the extra 30 pounds.

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">

```r
price_difference <- auction_data %>%
	separate(medium, sep="-", into=c("medium_min", "medium_max"), convert=TRUE) %>%
	separate(large, sep="-", into=c("large_min", "large_max"), convert=TRUE) %>%
	mutate(medium_midpoint = (medium_max + medium_min) / 2,
		large_midpoint = (large_max + large_min) / 2,
		difference = (large_midpoint * 125 - medium_midpoint * 95)/100) %>%
	select(date, difference)

price_difference %>%
	ggplot(aes(x=date, y=difference)) +
		geom_line(color="gray") +
		geom_smooth(span=0.1, se=FALSE) +
		labs(title="Depending on the time of year, it should cost you less than $20 to 50 to convert\na medium lamb to a large lamb (don't sell large lambs in January!)",
				subtitle="Difference in total price for a 95 and 125 pound lamb since 2015",
				caption="Data as reported by United Producers in Manchester, MI",
				x="Date",
				y="Difference in total price ($/animal)") +
		theme_light()
```

The difference in total price seems to be driven by the strong spring market for large lambs. This is very interesting to me beause it shows me a few things. First, if I have a lamb that will be 125 pounds in January, I probably should have sold it 6 weeks earlier as a medium lamb. Second, it shows me that aside from January, I need to spend less than $20 per lamb to get it to gain the extra 30 pounds for it to get into the large class.
</div>
