---
layout: post
title: "Candy Crush"
author: "PD Schloss"
date: 2002-03-26
time: 15:00 Eastern
blurb: "Let's explore people's favorite candies using functions from the tidyverse"
comments: false
---

For our inaugural Code Club, we will be exploring data collected by [Five Thirty Eight](https://fivethirtyeight.com/videos/the-ultimate-halloween-candy-power-ranking/) to determine what factors figure in to making a candy someone's favorite. You will be able to join the conversation using [this link](https://zoom.us/j/667635601?pwd=eGdBdTFpMjdVSXgrZjRXN2dzNDRnUT09) to use Zoom. The session should last an hour. Considering this is our first session, it may go a little longer as work out some of the kinks. Please be sure to see the [setup instructions](/code_club/setup-instructions) and [code of conduct](/code_club/code-of-conduct) before we get going.

If you would like to revisit Pat's introduction and the approach taken by several participants, you can watch this video (apologies for the video and audio quality!):

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/IpFOsMeDs9Q" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Assignment

Today's Code Club has three questions and a "homework" assignment. We'll be using the data that accompanied the Five Thirty Eight article that they were kind enough to [post to GitHub](https://github.com/fivethirtyeight/data/tree/master/candy-power-ranking). To load the tidyverse and the dataset, please run these lines of code in the R console. If you didn't already install the `tidyverse` package, you will need to follow the [setup instructions](/code_club/setup-instructions):

```R
library(tidyverse)

candy_data <- read_csv("https://raw.githubusercontent.com/fivethirtyeight/data/master/candy-power-ranking/candy-data.csv",
	col_types="clllllllllddd")
```

### Exercise 1

I'm a big fan of bite-sized candies - Peanut M&Ms, Skittles, Hot Tamales - mmmmmm. Within the pantheon of candies they included (N=85), I would like to know how common bite-sized candies are. I grabbed this code chunk. It gives me the answers that I'm looking for, but I don't understand the code. Run each line with the previous line (up to the the `%>%` or `+` at the end of the line) to see what is happening with each added line. Write a brief comment to the right of the `#` so that you can remember what is happening the next time you need to do something like this

```R
candy_data %>%	#Add comments here
	filter(pluribus) %>% #
	pivot_longer(
		cols=c(chocolate, fruity, caramel, peanutyalmondy, nougat, crispedricewafer, hard, bar, pluribus),
		names_to="type",
		values_to="answer") %>% #
	filter(answer) %>% #
	group_by(type) %>% #
	summarize(total = sum(answer)) #
```

### Exercise 2

It appears that among the bite-sized candies, the chocolatey (Peanut M&Ms) and fruity (Skittles) are the most common. Do people prefer bite sized candies with chocolate or fruity flavor? I know the code below works, but I accidentally sorted the lines and the code no longer works! There were originally three code chunks - one that builds a data frame containing only pluribus candies, one that reports some summary statistics about chocolatey and fruity bite-sized candies, and one that builds a strip chart showing a point for each type of candy. Unjumble the code to recreate the three code chunks. There are no missing or extra lines of code. Also, feel free to add comments!

```R
coord_cartesian(ylim=c(0,100)) +
filter(answer)
filter(pluribus & (chocolate | fruity)) %>%
geom_jitter(width=0.1) +
ggplot(aes(x=type, y=winpercent, color=hard)) +
group_by(type) %>%
labs(y="Contests won (%)", x=NULL, title="Bite-sized candies containing chocolate are preferred to candy without") +
pivot_longer(cols=c(chocolate, fruity), names_to="type", values_to="answer") %>%
scale_x_discrete(breaks=c("chocolate", "fruity"), labels=c("Chocolate", "Fruity")) +
summarize(median=median(winpercent), IQR=IQR(winpercent), N=n())
theme_classic()
pluribus_data %>%
pluribus_data %>%
pluribus_data <- candy_data %>%
```

### Exercise 3
Thanks! Now that you've fixed the code in Exercise 2, can you modify it to answer a new question for me? Are chocolate candies more expensive as a bar or as bite-sized pieces? Pick a third variable to color the data by. Copy and paste your code from Exercise 2 and edit it to help answer my question.


### Homework
Build your own strip plot using data using the `candy_data` data frame!


## Solution
<input type="button" class="hideshow">
<div markdown="1" style="display:none;">

```
### Exercise 1

candy_data %>%	#Spit out the data frame that we read in above for our pipeline
	filter(pluribus) %>% #Return those rows that contain bite-sized candies
	pivot_longer(
		cols=c(chocolate, fruity, caramel, peanutyalmondy, nougat, crispedricewafer, hard, bar, pluribus),
		names_to="type",
		values_to="answer") %>% #Concatenate all of the characteristic columns into two columns
	filter(answer) %>% #Remove those rows where a characteristic had been FALSE
	group_by(type) %>% #Arrange the data by characteristics
	summarize(total = sum(answer)) #Return the number of candies with each characteristic



## Exercise 2

pluribus_data <- candy_data %>%
	filter(pluribus & (chocolate | fruity)) %>%
	pivot_longer(cols=c(chocolate, fruity), names_to="type", values_to="answer") %>%
	filter(answer)

pluribus_data %>%
	group_by(type) %>%
	summarize(median=median(winpercent), IQR=IQR(winpercent), N=n())

pluribus_data %>%
	ggplot(aes(x=type, y=winpercent, color=hard)) +
	geom_jitter(width=0.1) +
	scale_x_discrete(breaks=c("chocolate", "fruity"), labels=c("Chocolate", "Fruity")) +
	coord_cartesian(ylim=c(0,100)) +
	labs(y="Contests won (%)", x=NULL, title="Bite-sized candies containing chocolate are preferred to candy without") +
	theme_classic()



## Exercise 3

pluribus_data <- candy_data %>%
	filter(chocolate & (pluribus | bar)) %>%
	pivot_longer(cols=c(pluribus, bar), names_to="type", values_to="answer") %>%
	filter(answer)

pluribus_data %>%
	group_by(type) %>%
	summarize(median=median(pricepercent), IQR=IQR(pricepercent), N=n())

pluribus_data %>%
	ggplot(aes(x=type, y=pricepercent, color=peanutyalmondy)) +
	geom_jitter(width=0.1) +
	scale_x_discrete(breaks=c("bar", "pluribus"), labels=c("Bar", "Bite-sized")) +
	coord_cartesian(ylim=c(0,1)) +
	labs(y="Price (fraction)", x=NULL, title="Chocolate candy bars tend to be more expensive than bite-sized chocolate candies") +
	theme_classic()
```

</div>


## Follow up
Once you have completed the Code Club, please complete [this survey](https://forms.gle/8a3dRbDsf4MYUX5a7).
