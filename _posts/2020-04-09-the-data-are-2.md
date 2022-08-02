---
layout: post
title: "The data is..."
author: "PD Schloss"
date: 2020-04-09
time: 15:00 Eastern
blurb: "The data are in and ... we need to analyze it/them"
comments: false
---

For our next Code Club, we will continue our exploration of data collected by [Five Thirty Eight](https://fivethirtyeight.com/features/elitist-superfluous-or-popular-we-polled-americans-on-the-oxford-comma/) that looked at how people feel about the [oxford comma](https://knowyourmeme.com/photos/946417) and whether the word ["data" is plural](https://en.wikipedia.org/wiki/Yes_(band)). [Last week](2020-04-02-the-data-are), we began to look at these data by renaming the column titles using the `rename` function and made the values in those columns easier to work with using the `recode` function. This week we will work with the `count` and `filter` functions to identify rows in the data frame that satisfy certain requirements. We will see how we can use these two functions to subset our `github` data frame from last week and characterize the survey responses for different groups of people. You will be able to join the conversation using [this link](https://zoom.us/j/593937827?pwd=ZUxNM3M4djdBVEM4VE5DKzlGZzhLZz09) to use Zoom. The session should last an hour. Please be sure to see the [setup instructions](/code_club/setup-instructions) and [code of conduct](/code_club/code-of-conduct) before we get going.

If you would like to revisit Pat's introduction and the approach taken by several participants, you can watch this video. If you were on the call, you'll notice a different Introduction. Pat forgot to press record the first time around:

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/sAwXtHv6sNA" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Prompt

Here's where we ended last week. Go ahead and copy this into a new R script file in RStudio

```R
library(tidyverse)

github <- read_csv("https://raw.githubusercontent.com/fivethirtyeight/data/master/comma-survey/comma-survey.csv") %>%
	rename(respondent=RespondentID,
		oxford_or_not="In your opinion, which sentence is more gramatically correct?",
		heard_of_oxford = "Prior to reading about it above, had you heard of the serial (or Oxford) comma?",
		care_about_oxford = "How much, if at all, do you care about the use (or lack thereof) of the serial (or Oxford) comma in grammar?",
		singular_or_plural = "How would you write the following sentence?",
		think_about_data = "When faced with using the word \"data\", have you ever spent time considering if the word was a singular or plural noun?",
		care_about_data = "How much, if at all, do you care about the debate over the use of the word \"data\" as a singluar or plural noun?",
		importance_of_grammar = "In your opinion, how important or unimportant is proper use of grammar?",
		gender = "Gender",
		age = "Age",
		household_income = "Household Income",
		education = "Education",
		location = "Location (Census Region)"
	) %>%
	mutate(oxford_or_not=recode(oxford_or_not, "It's important for a person to be honest, kind and loyal."="non_oxford",
						"It's important for a person to be honest, kind, and loyal."="oxford")) %>%
	mutate(singular_or_plural=recode(singular_or_plural, "Some experts say it's important to drink milk, but the data is inconclusive."="singular",
							"Some experts say it's important to drink milk, but the data are inconclusive."="plural"))
```

### `count`

Last week we saw the `count` function could be used to count the number of times categorical variable values appear. For example,

```R
github %>%
	count(oxford_or_not)
```

Generates a table telling us how many people chose the sentence with or without an Oxford comma. We can make the counts conditional on other variables, by including other variables separated by commas

```R
github %>%
	count(oxford_or_not, singular_or_plural)
```

This information is useful, but I find `count` to be useful for helping me to remember the values for a categorical variable. For example, I'd like to know what levels of education were included in the study.

```R
github %>%
	count(education)
```

A final point that is important, but easy to forget - the output of `count` is a new data frame that will have columns for the variables you are counting along with the number of cases. The other data in `github` will not be carried into the new data frame.


### `filter`

Today we're going to cover the `filter` function. `filter` allows us to select rows from a data frame for further downstream processing. For example, we could do the following to get those respondents that use the Oxford comma

```R
github %>%
	filter(oxford_or_not == "oxford")
```

You'll see that the original `github` variable had 1129 rows and this filtered version has 641. You'll also notice that the `oxford_or_not` column only has `oxford` in the first 10 rows. We can double check this using the `count` function from last week

```R
github %>%
	filter(oxford_or_not == "oxford") %>%
	count(oxford_or_not)
```

Let's step back and explain what's going on here. If we look at the argument for `filter`, we see `oxford_or_not == "oxford"`. This "asks" whether the value in the `oxford_or_not` variable for each row is `"oxford"`. The `==` is a logical equal, which asserts, "The left side is equal to the right side." If that statement is true, then the expression is `TRUE` if not, the expression is `FALSE`. `filter` evaluates this expression for every row and when the expression is `TRUE`, the row is retained. You may recall that in the Candy Crush code club we the line, `filter(answer)`. The `answer` variable was a logical variable where every value was a `TRUE` or `FALSE`. It's the same idea as our `oxford_or_not == "oxford"`, except `answer` already had the, um, answer.

A number of functions can be used to generate a logical (i.e. `TRUE` or `FALSE`) value. Not all of them are a fit for this dataset, we'll use `==` and `!=`. We've seen `==`, but what about `!=`? The `!` in logical operations means "not". So, `!=` means "not equal to". Let's try it out...

```R
github %>%
	filter(oxford_or_not != "oxford") %>%
	count(oxford_or_not)
```

That's effectively the same as

```R
github %>%
	filter(oxford_or_not == "non_oxford") %>%
	count(oxford_or_not)
```

Whether you use `==` or `!=` depends on how you think of your question.

Perhaps we want to know about people that use the Oxford comma and use "data" as a plural word. We have a few approaches to this question. Using what we already know, you could use two `filter` function calls

```R
github %>%
	filter(oxford_or_not == "oxford") %>%
	filter(singular_or_plural == "plural") %>%
	count(oxford_or_not, singular_or_plural)
```

The pipeline firsts filters based on whether the respondent used the Oxford comma and then on those who use data as a plural noun. We can simplify this to make a single `filter` function call

```R
github %>%
	filter(oxford_or_not == "oxford", singular_or_plural == "plural") %>%
	count(oxford_or_not, singular_or_plural)
```

That works! The `,` tells `filter` that both expressions have to evaluate as `TRUE`. I prefer to use `&` rather than the `,`

```R
github %>%
	filter(oxford_or_not == "oxford" & singular_or_plural == "plural") %>%
	count(oxford_or_not, singular_or_plural)
```

The reason I prefer to use the `&` (i.e. the AND operator) is because it allows us to generate more complex queries and it is a good partner to its opposite, `|`, the OR operator. Let's say I want a table of people who use the Oxford comma or who treat data as a plural noun

```R
github %>%
	filter(oxford_or_not == "oxford" | singular_or_plural == "plural") %>%
	count(oxford_or_not, singular_or_plural)
```

For the AND operator, ***both*** the left and right side of the `&` must be `TRUE` for the expression to be `TRUE`. For the OR operator, ***either*** the left or right side of the `|` must be `TRUE`  for the expression to be `TRUE`. That's why the output from the last expression doesn't have a row for `non_oxford` / `singular`.

Let's ask another slightly more complicated question. Let's generate a data frame of respondents that care about the Oxford comma "some" or "a lot" and use the Oxford comma. For these types of questions, it is helpful to use parentheses to organize our query. Remember that the stuff in parentheses is done first.

```R
github %>%
	filter(oxford_or_not == "oxford" & (care_about_oxford == "Some" | care_about_oxford == "A lot")) %>%
	count(oxford_or_not, care_about_oxford)
```

Finally, remember that we can put different columns in the `count` function than those that we filtered on. Among people that have heard of the Oxford comma, how much do people care about it?

```R
github %>%
	filter(heard_of_oxford == "Yes") %>%
	count(care_about_oxford)
```


## Exercises

1\. Which geographic region was the best represented in the survey?  
2\. How many respondents cared about grammar?  
3\. Among those respondents that cared about grammar, did they have a preference for the Oxford comma or using "data" as a plural noun?  
4\. Come up with your own question and answer it using `filter` and `count`  


## Solution
<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
1. Which geographic region was the best represented in the survey?

```R
github %>%
	count(location)
```

Pacific


2. How many respondents cared about grammar?

```R
github %>%
	filter(importance_of_grammar == "Somewhat important" | importance_of_grammar == "Very important")
```

1021

3. Among those respondents that cared about grammar, did they have a preference for the Oxford comma or using "data" as a plural noun?

```R
github %>%
	filter(importance_of_grammar == "Somewhat important" | importance_of_grammar == "Very important") %>%
	count(oxford_or_not, singular_or_plural)
```

4. Come up with your own question and answer it using `filter` and `count`

Are older people more likely to use singular or plural?

```R
github %>%
	count(age)

github %>%
	count(singular_or_plural)

github %>%
	filter(age == "45-60" | age == ">60") %>%
	count(singular_or_plural)
```

```
> github %>%
+ count(singular_or_plural)
# A tibble: 3 x 2
  singular_or_plural     n
  <chr>              <int>
1 plural               228
2 singular             865
3 NA                    36
>
> github %>%
+ filter(age == "45-60" | age == ">60") %>%
+ count(singular_or_plural)
# A tibble: 2 x 2
  singular_or_plural     n
  <chr>              <int>
1 plural                64
2 singular             226
>
```

Meh. how about the oxford comma?

```R
github %>%
	count(oxford_or_not)

github %>%
	filter(age == "45-60" | age == ">60") %>%
	count(oxford_or_not)
```

```
> github %>%
+ count(oxford_or_not)
# A tibble: 2 x 2
  oxford_or_not     n
  <chr>         <int>
1 non_oxford      488
2 oxford          641
>
> github %>%
+ filter(age == "45-60" | age == ">60") %>%
+ count(oxford_or_not)
# A tibble: 2 x 2
  oxford_or_not     n
  <chr>         <int>
1 non_oxford      148
2 oxford          142
```

Older people are less likely to use the Oxford comma

</div>
