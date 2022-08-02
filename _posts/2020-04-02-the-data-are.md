---
layout: post
title: "The data are..."
author: "PD Schloss"
date: 2020-04-02
time: 15:00 Eastern
blurb: "We've invited the rhinoceri, Washington and Lincoln to the next Code Club!"
comments: false
---

For our next Code Club, we will be exploring data collected by [Five Thirty Eight](https://fivethirtyeight.com/features/elitist-superfluous-or-popular-we-polled-americans-on-the-oxford-comma/) that looked at how people feel about two of the more contentious of Pat's Pedantic Points: the [oxford comma](https://knowyourmeme.com/photos/946417) and whether the word ["data" is plural](https://en.wikipedia.org/wiki/Yes_(band)). You will be able to join the conversation using [this link](https://zoom.us/j/592527970?pwd=SnBnWWRDVkVUS3BTbDZkWkxobjZsZz09) to use Zoom. The session should last an hour. Please be sure to see the [setup instructions](/code_club/setup-instructions) and [code of conduct](/code_club/code-of-conduct) before we get going. You should also check out the [previous session](2020-03-26-candy-crush) where we analyzed candy data. This go around we'll spend a little more time talking about how to prepare our data using the `rename` and `recode` functions from `dplyr`.

If you would like to revisit Pat's introduction and the approach taken by several participants, you can watch this video:

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/-C8cBbw7tq0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Prompt

To get started, copy and paste the lines in the code chunk below into a new R script in Rstudio and run them in the console

```R
library(tidyverse)

github <- read_csv("https://raw.githubusercontent.com/fivethirtyeight/data/master/comma-survey/comma-survey.csv")

github
```

```
# A tibble: 1,129 x 13
   RespondentID `In your opinio… `Prior to readi… `How much, if a…
          <dbl> <chr>            <chr>            <chr>           
 1   3292953864 It's important … Yes              Some            
 2   3292950324 It's important … No               Not much        
 3   3292942669 It's important … Yes              Some            
 4   3292932796 It's important … Yes              Some            
 5   3292932522 It's important … No               Not much        
 6   3292926586 It's important … No               A lot           
 7   3292908135 It's important … Yes              A lot           
 8   3292869879 It's important … Yes              A lot           
 9   3292863455 It's important … Yes              A lot           
10   3292860428 It's important … No               Not at all      
# … with 1,119 more rows, and 9 more variables: `How would you write the
#   following sentence?` <chr>, `When faced with using the word "data", have
#   you ever spent time considering if the word was a singular or plural
#   noun?` <chr>, `How much, if at all, do you care about the debate over the
#   use of the word "data" as a singluar or plural noun?` <chr>, `In your
#   opinion, how important or unimportant is proper use of grammar?` <chr>,
#   Gender <chr>, Age <chr>, `Household Income` <chr>, Education <chr>,
#   `Location (Census Region)` <chr>
```

### rename

We see from that output that some of the column names get truncated and are hard to read. We can use `colnames` to get a listing of the full column names

```R
colnames(github)
```

```
 [1] "RespondentID"  
 [2] "In your opinion, which sentence is more gramatically correct?"                                                           
 [3] "Prior to reading about it above, had you heard of the serial (or Oxford) comma?"                                         
 [4] "How much, if at all, do you care about the use (or lack thereof) of the serial (or Oxford) comma in grammar?"            
 [5] "How would you write the following sentence?"
 [6] "When faced with using the word \"data\", have you ever spent time considering if the word was a singular or plural noun?"
 [7] "How much, if at all, do you care about the debate over the use of the word \"data\" as a singluar or plural noun?"       
 [8] "In your opinion, how important or unimportant is proper use of grammar?"                                                 
 [9] "Gender"                                          
[10] "Age"                           
[11] "Household Income"                       
[12] "Education"                   
[13] "Location (Census Region)"                   
```

These column names are easy for humans to read, but pretty horrible for humans to retype or for computers to read. However you decide to name columns or variables, it is important to be consistent. It's a pain to forget whether you are using all lower case, all upper case, title case, etc. Do it the same way everytime and you won't have to think about it too much. Of course, other software and our collaborators, don't share our amazing sense of style! Sometimes we need to rename the columns. We want to keep our raw data raw, so we'll use a script to help clean up the column names.

Although much of this is a sense of personal style, there are some general guidelines to help you. Your column names should not have spaces in them, quotes (`"` or `'`), punctuation (e.g. `?`, `-`, `(`, `)`), or mixed capitalization. If you need a space, use an underscore (`_`). Also, try to make the column name brief, but descriptive. It can also be helpful to include the units in the column name (e.g. `height_cm`). These guidelines are to save you from problems with R, but also to help you make sure you keep things clear and consistent.

We can *rename* our columns after reading in a data frame using the rename function. The syntax is...

```R
rename(data_frame, new_name1="old name1", new_name2=old_name2, ...)

# or...

data_frame %>%
	rename(data_frame, new_name1="old name1", new_name2=old_name2, ...)
```

Note that if your old (or new) column name has a space in it, you will need to wrap the name in quotes. Also, if your column name has quotes in it, you will want to either use the opposite type of quote (single or double) to wrap the column name or insert a `\` before the quote mark within the phrase. Here's an example of how we could rename the `RespondentID` and `In your opinion, which sentence is more gramatically correct?` column names.

```R
rename(github, respondent=RespondentID, oxford_or_not="In your opinion, which sentence is more gramatically correct?")

# or...
github %>%
	rename(respondent=RespondentID,
		oxford_or_not="In your opinion, which sentence is more gramatically correct?")
```


### recode

We may also want to restate the values in a column so that they're easier to work with. For example, if we have mixed capitalization, common misspellings, Yes/No rather than TRUE/FALSE, non-sensical values (e.g. weight of 0 kg), etc.

```R
data_frame %>%
	mutate(
		recode_column=recode(recode_column, old_value1=new_value1, old_value2=new_value2, ...)
```

For our example, let's recode the `oxford_or_not` column to convert the sentence into whether or not it uses the Oxford comma.

```R
github %>%
	rename(respondent=RespondentID,
		oxford_or_not="In your opinion, which sentence is more gramatically correct?") %>%
	mutate(oxford_or_not=recode(oxford_or_not, "It's important for a person to be honest, kind and loyal."="non_oxford",
						"It's important for a person to be honest, kind, and loyal."="oxford"))
```

You'll notice we're using the `mutate` function, which is used to "mutate" a column. We're using the `oxford_or_not` column in the `recode` function to generate a new list of values and using those to replace the original `oxford_or_not` column. We could recode another column by adding a column to change within the same `mutate` function after we insert a comma between the two closing parentheses.


## Exercises

* Use `rename` to rename more of the columns in the data frame. Double check the result by adding `%>% colnames()` to the end of your pipeline.
* Use `recode` to recode the values in more of the columns
* Add the following chunk of code to our pipeline to get the fraction of people in the survey that would use the Oxford comma. Can you modify the code to see what fraction would use "data" as a singular vs. plural word?

	```R
	github %>%
		rename(respondent=RespondentID,
			oxford_or_not="In your opinion, which sentence is more gramatically correct?") %>%
		mutate(oxford_or_not=recode(oxford_or_not, "It's important for a person to be honest, kind and loyal."="non_oxford",
											"It's important for a person to be honest, kind, and loyal."="oxford")) %>%
		count(oxford_or_not) %>%
		mutate(percentage = 100 * n/sum(n))
	```

## Solution
<input type="button" class="hideshow">
<div markdown="1" style="display:none;">

### Exercise 1

Here are some possible values for `new_name` for each of the `old_name` values that I came up with...

|new_name | old_name|
|--------------------
| respondent | RespondentID |
| oxford_or_not | In your opinion, which sentence is more gramatically correct? |
| heard_of_oxford | Prior to reading about it above, had you heard of the serial (or Oxford) comma? |
| care_about_oxford | How much, if at all, do you care about the use (or lack thereof) of the serial (or Oxford) comma in grammar? |
| singular_or_plural | How would you write the following sentence? |
| think_about_data | When faced with using the word \"data\", have you ever spent time considering if the word was a singular or plural noun? |
| care_about_data | How much, if at all, do you care about the debate over the use of the word \"data\" as a singluar or plural noun? |
| importance_of_grammar | In your opinion, how important or unimportant is proper use of grammar? |
| gender | Gender |
| age | Age |
| household_income | Household Income |
| education | Education |
| location | Location (Census Region) |

I wrote the `rename` step as follows...

```R
github %>%
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
	) %>% colnames()
```

### Exercises 2 and 3
```R
github %>%
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
	mutate(singular_or_plural=recode(singular_or_plural, "Some experts say it's important to drink milk, but the data is inconclusive."="singular",
							"Some experts say it's important to drink milk, but the data are inconclusive."="plural")
	) %>%
	count(singular_or_plural) %>%
	mutate(percentage = 100 * n/sum(n))
```

</div>

## Follow up
Once you have completed this Code Club, please complete [this survey](https://forms.gle/pGQQckwW66m9vBrA9).
