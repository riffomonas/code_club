---
layout: post
title: "Find the candy that I like..."
author: "PD Schloss"
date: 2020-04-16
time: 15:00 Eastern
blurb: "The Easter-induced candy stupor as faded, now to analyze the data"
comments: false
---

For our next Code Club, we will continue our exploration of data collected by [Five Thirty Eight](https://fivethirtyeight.com/videos/the-ultimate-halloween-candy-power-ranking/) that looked at how people feel about different types of candy. [In the first code club](2020-03-26-candy-crusy), we initially played with the data. Last week, we began to use the `filter` function to generate logical (or boolean) queries to identify rows from the data that met our criteria. This time we'll focus on using `filter` with quantitative data and we'll see how to use it with the `group_by` and `summarize` functions.

You will be able to join the conversation using [this link](https://zoom.us/j/96466835271?pwd=dGljcGc5clFyeVI5ZjJsdHFOQWo0UT09) to use Zoom. The session should last an hour. Please be sure to see the [setup instructions](/code_club/setup-instructions) and [code of conduct](/code_club/code-of-conduct) before we get going.

If you would like to revisit Pat's introduction and the approach taken by several participants, you can watch this video. If you were on the call, you'll notice a different Introduction. Pat forgot to press record the first time around (again):

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/-USF9PFOvvY" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Prompt

In our inaugural code club we loaded data that described people's preferences for each of 85 different types of candy. The folks at Five Thirty Eight characterized each of the candies by their attributes including things like whether the candy has chocolate, nougat, caramel, and so forth. They also created indices to rate the relative prices and amount of sugar in each candy. In that code club, I had participants generate a figure that showed whether chocolate candies are more expensive as a bar or as bite-sized pieces. I'd like to use that question to build upon the discussion we had last week using `filter` and to expand the discussion to calculate statistics for different groups of data using `group_by` and `summarize`. Let's load the data and get going!

```R
library(tidyverse)

candy_data <- read_csv("https://raw.githubusercontent.com/fivethirtyeight/data/master/candy-power-ranking/candy-data.csv",
	col_types="clllllllllddd")
```


### `filter` revisited

You'll recall that `filter` identifies rows in a data frame that satisfy some logical question. If the answer to the question is `TRUE`, then that row is returned. These candy data have a number of variables that are already `TRUE` or `FALSE` values.

```R
candy_data
```

If you look under the "chocolate" heading, you'll see <lgl> which is short for logical. You'll also see that column (and others) have `TRUE` and `FALSE` as their values. Since these already evaluate to a logical value, I can do the following to get all of the candies that have chocolate in them

```R
candy_data %>%
	filter(chocolate)
```

While you can do

```R
candy_data %>%
	filter(chocolate == TRUE)
```

It's a bit redundant and not necessary. Regardless of the approach, we see that 37 of the candies have chocolate in them. Of course, we could have figured that out with the `count` function that we discussed earlier

```R
candy_data %>%
	count(chocolate)
```

What the `filter` command gets us is a new data frame that only contains candies with chocolate. Before moving on to discussing `group_by` and `summarize`, I want to show you a few other things we can do with `filter`.

So far we've seen that we can search for specific values using the `==` and `!=` operators and that we can combine queries with the `&` and `|` operators. The `==` and `!=` operators work well when we're trying to select for specific values. Sometimes, we want a range of values. Let's say we want a new data frame that contains all of the candies that have a price index greater than 0.5, we could do the following

```R
candy_data %>%
	filter(pricepercent > 0.5)
```

Similarly, if we wanted all of the low sugar candies we could do

```R
candy_data %>%
	filter(sugarpercent < 0.33)
```

And building upon what we did last week, we can combine the queries with a `&` to get the expensive low sugar candies or with a `|` to get the candies that are expensive or low in sugar.

```R
candy_data %>%
	filter(pricepercent > 0.5 & sugarpercent < 0.33) #12 rows

candy_data %>%
	filter(pricepercent > 0.5 | sugarpercent < 0.33) #64 rows
```

Beyond `>` and `<`, you can also use `>=` and `<=` to ask if things are "greater than or equal to" or "less than or equal to" a specific value.


### `group_by` and `summarize`

The `group_by` and `summarize` functions generally go together. They allow us to group our data by some variable and then to perform some summary operation for each group. Returning to our question of whether candy with chocolate is more expensive as a bar or in bite-sized form we can do the following

```R
candy_data %>%
	filter(chocolate == TRUE) %>%
	group_by(bar) %>%
	summarize(mean_price = mean(pricepercent))
```

We could also report back the standard deviation and number of candies in each group by adding them to the list of arguments in `summarize` separated by a comma

```R
candy_data %>%
	filter(chocolate == TRUE) %>%
	group_by(bar) %>%
	summarize(mean_price = mean(pricepercent), sd_price = sd(pricepercent), n=n())
```

Although we should apply a statistical test, it does appear that candy with chocolate is more expensive in bar form. Other functions that work well with `summarize` include `median`, `min`, `max`, `IQR`, `range`, and any function that returns a single value.

Perhaps we'd like to partition our data frame by two variables. Say we want to look at all candy to determine whether non-chocolate candy is also more expensive as a bar than as bite-sized. We could modify our earlier code chunk

```R
candy_data %>%
	filter(!chocolate) %>% # the ! turns TRUE to FALSE and FALSE to TRUE
	group_by(bar) %>%
	summarize(mean_price = mean(pricepercent))
```

If we want both chocolaty and non-chocolate candy data in the same data frame, we will need to drop the `filter` function and modify the `group_by` function. We can add other groups to partition the data by in the `group_by` function separated by commas

```R
candy_data %>%
	group_by(chocolate, bar) %>%
	summarize(mean_price = mean(pricepercent))
```

Sure enough, candy as a bar is more expensive regardless of whether it has chocolate in it!

One final trick to show you is that we can add a logical question to our `group_by` argument

```R
candy_data %>%
	group_by(sugarpercent > 0.50) %>%
	summarize(mean_price = mean(pricepercent))
```

Who knew? Candy with more sugar is more expensive.


## Exercises

1\. How many of the candies that won more than 75% of their matchups had chocolate?  

2\. Do fruity candies have a different average price than non-fruity candies?  

3\. How do the prices of the more favored candies compare to those that are less favored?  

4\. Come up with your own question to answer with the functions we've discussed today  


## Solutions
<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
1\. How many of the candies that won more than 75% of their matchups had chocolate?

```R
candy_data %>%
	filter(winpercent > 0.75) %>%
	count(chocolate)
```

There's more non-chocolate than chocolate containing candy in the dataset


2\. Do fruity candies have a different average price than non-fruity candies?

```R
candy_data %>%
	group_by(fruity) %>%
	summarize(mean_price = mean(pricepercent), sd_price=sd(pricepercent), n=n())
```

Fruity candy tends to be cheaper than candy without fruity flavor.


3\. How do the prices of the more favored candies compare to those that are less favored?

```R
candy_data %>%
	group_by(winpercent > 50) %>%
	summarize(mean_price = mean(pricepercent), sd_price=sd(pricepercent), n=n())
```

The more favored candies are more expensive.


4\. Come up with your own question to answer with the functions we've discussed today

Does price differ by sugar content?

```R
candy_data %>%
 	group_by(sugarpercent > 0.5) %>%
	summarize(mean_price = mean(pricepercent), sd_price=sd(pricepercent), n=n())
```

High sugar candy tends to be more expensive than low sugar candy. Perhaps confounded with people's preference?

```R
candy_data %>%
 	group_by(sugarpercent > 0.5, winpercent > 50) %>%
	summarize(mean_price = mean(pricepercent), sd_price=sd(pricepercent), n=n())
```

Maybe people just like good candy.
</div>
