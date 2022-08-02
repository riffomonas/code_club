---
layout: post
title: "Comparing lamb prices"
author: "PD Schloss"
date: 2020-06-18 11:55
blurb: "Do the prices for different classes of lamb move together? We'll use <code>pivot_longer</code>"
comments: false
---

In [last week's code club](2020-06-11-predicting-lamb-futures) we saw that large lambs are more valuable in the spring and that if aged sheep have any value, it's typically in January. But what about the other classeses of sheep? As a producer I have the choice of determining when to take lambs to the butcher. In southeastern Michigan there is a large Muslim community that eats a lot of lamb. They generally prefer lambs that are between 60 and 80 pounds. Other buyers prefer large lambs to get more meat per cut. All of this means that I have many options for how to market my lambs. Since we produce lambs throughout the year, I'd like to know whether there is an ideal time to market each classes of lambs or whether they all track each other giving more value in the Spring. That's the question we'll answer for today's episode of Code Club!

For today's Code Club, we'll build off of what we have done for the past few weeks. Don't worry if you missed those episodes. We'll start with the block of code that we used last week and see how we can modify what we did for large lambs to look at all lamb classes. To do this, we'll need to learn about the `pivot_longer` function from the `tidyr` package, which is part of the `tidyverse`. This function is related to `gather`, which is the older version of `pivot_longer`. Know that if you see people mention using `gather` that it's ok to keep using that function, but the package maintainers are encouraging people to pivot to `pivot_longer`. As always, don't watch the video straight through without firing up RStudio and trying the code and exercises yourself! Please be sure to see the [setup instructions](/code_club/setup-instructions) before you get going.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/J1TnSoSuAsg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Prompt

Today we'll be using the `tidyverse` and the [`googlesheets4`](https://googlesheets4.tidyverse.org) packages for reading my googlesheet. If you haven't already, you'll need to instal it as [described previously](2020-06-04-eat-more-lamb). Here is the code chunk we used [last week](2020-06-11-predicting-lamb-futures) that read in the data and cleaned up the column names.


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
```

To calculate the midpoint price for the large lambs, we used this block of code

```r
large_prices <- auction_data %>%
	separate(large, sep="-", into=c("min", "max"), convert=TRUE) %>%
	select(date, min, max)
```

Finally, we plotted it with this

```r
large_prices %>%
	mutate(mid_point = (min + max) / 2) %>%
	ggplot(aes(x=date, y=mid_point)) +
		geom_line(color="gray") +
		geom_smooth(span=0.1, se=FALSE) +
		labs(title="Large lambs have their highest value in April and May",
			subtitle="Prices for lambs between 106 and 130 pounds",
			caption="Data as reported by United Producers in Manchester, MI",
			x="Date",
			y="Price ($/100 lbs)") +
		theme_light()
```

### What are tidy data?

If you look at `auction_data` you will see that there are columns for each type of sheep and within each of those columns the prices are indicated as a range with a hyphen. Last week we used `separate` to create a `min` and `max` column for an individual classes. This structure for `auction_data` is fine if we only want to plot one classes. But what about our goal of plotting all of the classeses against each other? The problem with `auction_data` is that it is not tidy for our purposes.

We've talked a lot about the "tidyverse" over the past weeks, but we've never really discussed what it means. The general idea is that each column should represent a different variable and each row should represent a different thing being measured by those variables. Another way of thinking about it is to ask what columns you would map to different aesthetics if you were going to plot the data. Last week, the `date` column goes to the x-axis and `mid_point` goes to the y-axis. To compare multiple classeses, we'll want to map the different classeses to the color of the line. This means that we'll need the classes names as a column. You can think of what we want as copying and pasting each classes column to the end of the other to make a long, rather than wide, data frame. To achieve this, we will use `pivot_longer`. The other nice thing about this is that once we put all of our classeses into a single column, we can use `separate` to pull apart the min and max prices and then get the midpoint using mutate. We've seen all of this before, except for using `pivot_longer`.


### Tidy-ing our data

The `pivot_longer` function has a number of different options. We'll only use a few of them to answer today's question. The basic syntax that we need is to provide `pivot_longer` with the data we want to tidy (we can do this through the pipe), the columns we want to pivot into the longer format (`cols`), a string that names the new column of names that were previously the column names (`names_to`), and a string that names the new column of values that were previously in the columns (`values_to`). Let's test it out using a few columns from `auction_data`.

```r
auction_data %>%
	pivot_longer(cols = c(small, medium, large, extra_large), names_to="classes", values_to="price_range")
```

Writing out all of the names is a pain. As an alternative, we can also tell `pivot_longer` what we don't want to include.

```r
auction_data %>%
	pivot_longer(cols = -c(date, total), names_to="classes", values_to="price_range")
```

Nice! Now we have a tidy data frame. We don't need the `total` column in there, but it isn't doing any harm so we can leave it.


### Integrating our tidy data into the rest of the pipeline

The next thing we want to do is to separate the `price_range` column into two columns. We also want to make sure that those prices are converted from characters to numeric data and we'll also want to calculate the midpoint. I'll save this as `tidy_auction_data`

```r
tidy_auction_data <- auction_data %>%
	pivot_longer(cols = -c(date, total), names_to="classes", values_to="price_range") %>%
	separate(price_range, sep="-", into=c("min", "max"), convert=TRUE) %>%
	mutate(midpoint = (min + max) / 2)
```

If you look at the output of this code chunk think about what columns we want to map to each aesthetic. Last week we talked about the x and y aesthetics. If you look at the `geom_line` documentation, you'll see other aesthetics including the color aesthetic. Let's map `date` to x, `midpoint` to y, and `classes` to color

```r
tidy_auction_data %>%
	ggplot(aes(x=date, y=midpoint, color=classes)) +
		geom_line()
```

Nice! Well, not really, but we have a plot with all of the sheep midpoint prices shown for the past 5 years. Let's add a `filter` line to the last code chunk to look at the small, medium, large, and extra large lambs

```r
tidy_auction_data %>%
	filter(classes == "small" | classes == "medium" | classes == "large" | classes == "extra_large") %>%
	ggplot(aes(x=date, y=midpoint, color=classes)) +
		geom_line()
```

This is really cool (to me!). I'm going to send this to a friend of mine to see if she agrees with my insights. Before I do that and before we move on to the exercises, let's clean the plot up to make it a bit more presentable

```r
tidy_auction_data %>%
	filter(classes == "small" | classes == "medium" | classes == "large" | classes == "extra_large") %>%
	ggplot(aes(x=date, y=midpoint, color=classes)) +
		geom_line() +
		labs(title="Small lambs have the highest price, but\nall lambs peak in Spring",
			x="Date",
			y="Price ($/100 lbs)",
			caption="Data from United Producers in Manchester, MI") +
		theme_light()
```

Interesting. There are a few things going on here. First, it is a little hard to see because the data are noisy, but it appears that the smaller lambs peak earlier in the year (Passover and Easter) followed by the larger weight classeses. Small lambs do well in the first three months of the year. Second, the price per pound drops as the size of the lamb increases. Third, there doesn't really seem to be a difference in price between large and extra large lambs. Incidentally, the small weight classes appears to have been split into the feeder lamb and new crop classeses in 2019. There's a lot more we could do with this plot, but I don't want to get too carried away!


## Exercises

1\. The second page of my "lamb_prices" workbook is "holidays". It is a grid that has a number of holidays where people tend to eat lamb as rows and years as columns. The values are the actual dates. Tidy this "holidays" tab to have columns titled `holiday`, `year`, and `date`. Make sure that the `date` column is the "date" format and not "dttm".

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```R
read_sheet("https://docs.google.com/spreadsheets/d/1_quMjJRBHDLQSmWQouzzyi1DOejAtCZnAeesdVyRWiQ/edit#gid=1467293328",
		sheet="holidays",
		col_type = "cDDDDDDD") %>%
	pivot_longer(-holiday, names_to="year", values_to="date")
```
</div>


2\. The data in the final plot we made was pretty noisy. What happens if you use `geom_smooth` in place of `geom_line`?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">

```r
tidy_auction_data %>%
	filter(classes == "small" | classes == "medium" | classes == "large" | classes == "extra_large") %>%
	ggplot(aes(x=date, y=midpoint, color=classes)) +
		geom_smooth(span=0.1, se=FALSE) +
		labs(title="Small lambs have the highest price, but\nall lambs peak in Spring",
			x="Date",
			y="Price ($/100 lbs)",
			caption="Data from United Producers in Manchester, MI") +
		theme_light()
```
</div>


3\. It was never quite clear what the difference between a small lamb (40-65 lbs), new crop lamb, or feeder lamb was. In 2019, they stopped reporting prices for small lambs. My sense is that new crop lambs are small lambs that are ready for slaughter and the feeder lambs need more time to grow to one of the larger categories. Create a plot comparing the small, feeder lamb, and new crop columns. Which category most resembles the small lamb prices?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```r
tidy_auction_data %>%
	filter(classes == "small" | classes == "feeder_lambs" | classes == "new_crop") %>%
	ggplot(aes(x=date, y=midpoint, color=classes)) +
		geom_smooth(span=0.1, se=FALSE) +
	labs(title="The small lamb class was replaced by\nthe new crop category",
			x="Date",
			y="Price ($/100 lbs)",
			caption="Data from United Producers in Manchester, MI") +
		theme_light()
```

</div>
