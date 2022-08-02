---
layout: post
title: "When is she due?"
author: "PD Schloss"
date: 2020-05-28 11:55
blurb: "She's expecting! We'll use the <code>lubridate</code> package to figure out the due date."
comments: false
---

Breeding animals to produce offspring that we can market for meat is a big part of what we do on the farm. We breed sheep, pigs, and cows. If my 6 year old gets his way, we'll be consistently breeding turkeys as well. Breeding can be as easy as putting the males in with the females and letting nature take its course. Unfortunately, this can give us "oops" babies that arrive unexpectedly during a heavy Michigan snow. To have more control, we selectively breed our animals at certain dates to optimize the females' productivity and to market animals at certain times of the year for meet. As I'm planning my breeding program, I google the gestations times for each species of animal. These searches generally pull up a table that looks something like [this](https://nationalswine.com/resources/docs/Swine-Gestation-Table.pdf)

<img alt="pig gestation calendar" src="https://nationalswine.com/resources/docs/Swine-Gestation-Table.pdf" width="60%" style="margin: 0 auto; display: block"/>

You read this table by finding the month the sow is bred in (e.g. May), read over to the date (e.g. 28) and then look down a row in the same column to find when the sow will farrow (e.g. September 19). A handy heuristic to remember the gestation time for a sow is "3 months, 3 weeks, 3 days". That's not *exactly* right, but its within a day or two and is close enough for most farmers purposes. Looking up a date or two isn't so bad. Perhaps I want to do some more long term planning like planning a sow's year so that she farrows twice a year, 6 months apart.

To answer these types of questions, we'll learn about the functions in the `lubridate` package. By the end of today's Code Club we'll be much closer to planning out our breeding program for the pigs, sheep, and cows on my farm. Don't watch the video straight through without firing up RStudio and trying the code and exercises yourself! Please be sure to see the [setup instructions](/code_club/setup-instructions) before you get going.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/zyGUKNpBeiI" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Prompt

The `lubridate` package is installed with the `tidyverse` package, but it isn't loaded with `tidyverse` so we need to run the `library` function on it so we can use its functions.

```R
library(tidyverse)
library(lubridate)
```

### ISO 8601

The first topic to quickly discuss is how we format dates. Every country in the world uses the wrong format to represent a date. Before proceeding, write down today's date. I bet you came up with things like "May 28, 2020", "28 May 2020", "5/28/2020", "28/5/2020", "5/28/20" or some variation on these. If we were in January, then you might use "Jan" or "January" to represent the date. All of this becomes a challenge on a date like February 3rd. If we write the date as "2/3/2020", without additional information, it is unclear whether that is February 3rd or March 2nd. To get around this problem, [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) was created. The standard date format would write today's date as "2020-05-28" or "YYYY-MM-DD" - the four digit year, two digit month, and two digit day; each value separated by a hyphen. There are functions in `lubridate` that you can use to use other formats, but that's not the part of `lubridate` that I want to focus on today. Besides, why not emphasize good practices?! We can write a date as

```R
today <- date("2020-05-28")
```

We don't always need the `date` function because the `lubridate` functions generally do a good job of converting the string, "2020-05-28" to a date before doing its magic if the string is provided as an argument. We can also get today's date using the `now` function

```R
now()
```


### Adding days to a date

About a month ago on April 25th, I put two of my rams, "Big Tex" and "Pretty Boy" into the same pasture as a group of 27 ewes. My plan is to remove the rams from this group on May 30th. I need to know when I can expect the lambs will be born. This would be a relatively simple task for a gestation table like the one I showed above. The `lubridate` package has a set of functions including `years`, `months`, `weeks`, and `days` that will add that many units on to a date (it also has functions for dealing with time). A google search shows me that lamb gestation is generally between 142 and 152 days, so we'll go with 147 days. With this information, I can answer my question like this

```R
date("2020-04-25") + days(147)
date("2020-05-30") + days(147)
```

I should expect my lambs to be born between September 15th and October 20th this fall. Perhaps I should start expecting them around September 10th to be safe.


### Arithmetic with dates

As I said above, a handy heuristic to remember the gestation time for a sow is "3 months, 3 weeks, 3 days". That's not *exactly* right, but its within a day or two and is close enough for the actual gestation period length of 114 days. We saw our boar, Wilbur, mount our sow, TT, on April 7th. When should we expect her to farrow (i.e. deliver piglets)? Hopefully, you can see what I did above and generalize to this problem.

```R
breed_date <- date("2020-04-07")
breed_date + months(3) + weeks(3) + days(3)
```

This tells me that we can expect piglets to be born on July 31st. The actual gestation period is 114 days. What would that date be?

```R
date(breed_date) + days(114)
```

July 30th. Pretty close, eh?


Returning to the heuristic, how many days are in "3 months, 3 weeks, 3 days"? We could do this by manually adding the days in each month and in a week, but that wouldn't help us to learn `lubridate` very well. Also, some months have 30 days, some 30, and one usually has 28 days, but sometimes has 29. The lubridate `year`, `month`, and `days` functions has all that built in. We can find the number of days in the heuristic for any day of the year by subtracting out the breeding date.

```R
date(breed_date) + months(3) + weeks(3) + days(3) - date(breed_date)
```

Alternatively, we could use `difftime` by giving it the start and end dates and desired output units as arguments

```R
due_date <- breed_date + months(3) + weeks(3) + days(3)
difftime(due_date, breed_date, units="days")
```

That shows us a gestation period of 115 days, which as we've already seen is a day longer than the "true" gestation period. Let's try a different breeding date

```R
breed_date <- date("2020-02-07")
due_date <- breed_date + months(3) + weeks(3) + days(3)
difftime(due_date, breed_date, units="days")
```

114 days - we have a leap year this year. It would have been 113 days if we chose "2019-02-07" as the breeding date.

```R
breed_date <- date("2020-03-07")
due_date <- breed_date + months(3) + weeks(3) + days(3)
difftime(due_date, breed_date, units="days")
```

The heuristic is only a day or two longer than the actual breeding date and works pretty well when we don't have easy access to R or a gestation table!


### Back dating breeding times

We once had a couple of cows calve in January. It wasn't much fun. We don't have enough barn space for them and they're a lot harder to handle than a ewe or a sow. I'd like to make sure that I don't have any cows born before March 1 of 2021. If a cow's gestation period is 283 days, what's the earliest I should consider breeding them? This question isn't really much different from figuring out the due date from the breeding date. Instead of using addition, we can use subtraction.

```R
due_date <- date("2021-03-01")
due_date - days(283)
```

For this year, it looks like I'm safe to breed the cows since we're past May 22nd.

Hopefully, you feel more comfortable using some of the lubridate functions that allow us to add or subtract time from a date. Are you ready to answer some questions using these new skills?


## Assignment

1\. How many days were the rams in with the ewes?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```R
difftime("2020-05-30", "2020-04-25")

# or

date("2020-05-30") - date("2020-04-25")
```

Ewes ovulate every 17-19 days so this gives each ewe two times to be bred.
</div>


2\. We want piglets born after January 10 so my kids can have pigs to show for our local 4-H fair at the end of July. When should we expose the sows to the boar?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```R
due_date <- date("2020-01-10")
due_date - days(114)
```

September 18th.
</div>


3\. After the ewes that are being exposed have their lambs, they will nurse their lambs for 9 weeks after their due date. I then give them a recovery period to gain some weight before being bred again. If I want the ewes to deliver lambs every 8 months, how much recovery time should I give the ewes?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```R
first_breed_date <- date("2020-04-25")
first_due_date <- breed_date + days(147)
first_wean_date <- first_due_date + weeks(9)

second_due_date <- first_due_date + months(8)
second_breed_date <- second_due_date - days(147)

recovery_period <- difftime(second_breed_date, first_wean_date)
```

I can give them just over a month to recover before trying to breed them again. Of course, if there are ewes that use the full 35 days to get bred, their lambs will be quite a bit younger at the time of weaning. As a management strategy, we might decide it would be better to only give them 18 days to get bred if we have a lot of younger lambs at weaning time.
</div>


4\. The "3 months, 3 weeks, 3 days" heuristic is pretty catchy. Can you come up with a similar heuristic for sheep and cows?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
Let's start with cows at 283 days

```R
breed_date <- now()
due_date <- breed_date + months(9) + days(9)
difftime(due_date, breed_date) - 283
```

9 months and 9 days works pretty nicely for cow!

Now for sheep at 147 days

```R
breed_date <- now()
due_date <- breed_date + months(5) - days(5)
difftime(due_date, breed_date) - 147
```

5 months minus 5 days works well for sheep!
</div>

