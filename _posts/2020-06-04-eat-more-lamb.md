---
layout: post
title: "Eat more lamb!"
author: "PD Schloss"
date: 2020-06-04 11:55
blurb: "Is lamb production in Michigan on the rise? Using <code>googlesheets4</code> to access data for southern Michigan"
comments: false
---

On our farm I have about 40 ewes - female sheep that we keep year after year to produce more sheep. Most sheep are born between January and May and sold in the following fall. Those animals that are under a year old are called lambs and are generally sold when they're 60 to 130 pounds. This is generally what people eat. In the US, anything older is considered "mutton" and has an undeserved bad reputation associated with its taste. Because mutton was often sold as lamb, lamb consumption really fell throughout the 1900's relative to other animal proteins like beef, pork, and chicken. Surprisingly, most of the lamb that Americans eat is actually from Australia and New Zealand, not the United States. To counteract all of these trends, the lamb industry has made a big push for producers to increase the size of their flocks to grow the industry. Has this been effective? What effect has COVID-19 had on the number of lambs sold? Over the past 5 years, [my local livestock auction](https://www.uproducers.com/livestock-marketing/market-results/) in [Manchester, MI](https://goo.gl/maps/jJgd8vRDXP7HGEL59) has been posting the number of lambs they have sold along with the price range for various sized lambs and older sheep. When the new data are posted each week, I copy the data into a [Google sheet (it's publically viewable)](https://docs.google.com/spreadsheets/d/1_quMjJRBHDLQSmWQouzzyi1DOejAtCZnAeesdVyRWiQ/edit?usp=sharing) so that I can look for trends in things like the numbers of lambs sold and when I can get the best prices. The USDA also posts these types of data for larger markets. Because I live about 30 minutes from Manchester and they cater to a unique blend of traditional and Muslim markets, I'm most interested in the local data.

For today's Code Club, we'll use the `googlesheets4` package to read that spreadsheet and we'll see how we can use functions from the `ggplot2` package to plot the data. I have become a big fan of Google sheets because I can look at any of my sheets on my phone and I can enter data on the phone too. Also, if you use Google forms, you can export a Google sheet from the form. This can be handy if you're collecting data in the field or from research subjects and you want to constrain the types of answers they give. That we can use R to analyze the data makes it all the better! Don't watch the video straight through without firing up RStudio and trying the code and exercises yourself! Please be sure to see the [setup instructions](/code_club/setup-instructions) before you get going.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/mFrTG0LMMi8" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Prompt

Today we'll use the `tidyverse` package, but we'll also use the [`googlesheets4`](https://googlesheets4.tidyverse.org) package. We'll need to install it. You may also need to install `httpuv` to get `googlesheets4` to work correctly


```R
library(tidyverse)

install.packages("googlesheets4")
install.packages("httpuv")
library(googlesheets4)
```

### Reading Google sheets

Reading in a Google workbook is much like reading in a csv or xlsx file. We use `read_sheet` and give it the URL for the workbook we want to read in. The URL for my workbook containing the number of lambs sold is:

```
https://docs.google.com/spreadsheets/d/1_quMjJRBHDLQSmWQouzzyi1DOejAtCZnAeesdVyRWiQ/edit#gid=1467293328
```

To read it in, we use `read_sheet`

```r
read_sheet("https://docs.google.com/spreadsheets/d/1_quMjJRBHDLQSmWQouzzyi1DOejAtCZnAeesdVyRWiQ/edit#gid=1467293328")
```

When you run this command you will get a prompt either asking you how to access your Google account. Mine looks like this

```
> read_sheet("https://docs.google.com/spreadsheets/d/1_quMjJRBHDLQSmWQouzzyi1DOejAtCZnAeesdVyRWiQ/edit#gid=1467293328")
The googlesheets4 package is requesting access to your Google account. Select a pre-authorised account or enter '0' to obtain a new token. Press Esc/Ctrl + C to abort.

1: pdschloss@gmail.com
2: pschloss@umich.edu

Selection:
```

If you select 0, it will open a web page dialog to log in. Go ahead and do that. Note that if you don't have or want a Google account, you can get this particular file by running `gs4_deauth()` before `read_sheet`. This approach works here, because anyone that has the link to the work sheet has rights to view the file.

By default, running `read_sheet` returns the first sheet from the workbook. If we want a different sheet from the workbook, we need to use the `sheet` argument.

```r
read_sheet("https://docs.google.com/spreadsheets/d/1_quMjJRBHDLQSmWQouzzyi1DOejAtCZnAeesdVyRWiQ/edit#gid=1467293328", sheet="holidays")
```

Note that there is an accompanying [Google drive](https://googledrive.tidyverse.org/index.html) package that helps you to access your Google drive so that you can interact with it, much like you can with your local file system.

Looking at the `numbers_and_prices` sheet from the workbook, you might notice that there is a column named `...11`. What is this? If you go back to the Google sheet you'll notice that there is information in Column K that has information about some of the dates or other notes about what may be connected with that week's data. I'd like to exclude that data. I can get a specific range of cells using the `range` argument. To get columns A through J, I can use `range="A:J"`

```r
read_sheet("https://docs.google.com/spreadsheets/d/1_quMjJRBHDLQSmWQouzzyi1DOejAtCZnAeesdVyRWiQ/edit#gid=1467293328", range="A:J")
```

Similarly, we can look at a subset of cells by also including the row number.

```r
read_sheet("https://docs.google.com/spreadsheets/d/1_quMjJRBHDLQSmWQouzzyi1DOejAtCZnAeesdVyRWiQ/edit#gid=1467293328", range="A1:B10")
```

This gets us the data in the rectangle between cells A1 and B10. Let's focus in the date and the number of lambs sold that week

```r
read_sheet("https://docs.google.com/spreadsheets/d/1_quMjJRBHDLQSmWQouzzyi1DOejAtCZnAeesdVyRWiQ/edit#gid=1467293328", range="A:B")
```

We can now see a few weird things about our new data frame. Namely, the `Date` column has the date and time and is of type `dttm` rather than `date` and the `Total` column is of type `list` and each of the first 10 rows (at least) has `<dbl [1]>` rather than a number. To solve these problems, we need to help `read_sheet` by telling it the type of data in each of the columns and the characters we are using to indicate missing data (i.e. `NA`). We can tell `read_sheet` the columns types by using a single character code for various data types. A date would be `D`, double precision number  would be `d`, numeric would be `n`, logical would be `l`, etc. You can get a full list in the "Column specification" section of the output from running `?read_sheet`. We'll add `col_types="Dd"`

```r
read_sheet("https://docs.google.com/spreadsheets/d/1_quMjJRBHDLQSmWQouzzyi1DOejAtCZnAeesdVyRWiQ/edit#gid=1467293328", range="A:B", col_types="Dd")
```

This command gives us the expected output, but does tell us that there are 45 warning messages. Runnig `warnings()` at the prompt shows us the message `In .Primitive("as.double")(x, ...) : NAs introduced by coercion` repeated 45 times. We can get rid of this by using the `na` argument to indicate the string that indicates NA values.

```r
read_sheet("https://docs.google.com/spreadsheets/d/1_quMjJRBHDLQSmWQouzzyi1DOejAtCZnAeesdVyRWiQ/edit#gid=1467293328", range="A:B", col_types="Dd", na="NA")
```

There you go! We've read in data from a Google sheet and gotten it properly formatted.


### Writing Google sheets

Perhaps we'd like to now write this two column data frame back to our own Google drive. We can do this by using `write_sheet`

```r
sheep_numbers <- read_sheet("https://docs.google.com/spreadsheets/d/1_quMjJRBHDLQSmWQouzzyi1DOejAtCZnAeesdVyRWiQ/edit#gid=1467293328", range="A:B", col_types="Dd", na="NA")

write_sheet(sheep_numbers)
```

If you then go look in your Google drive, you'll see a file that has a rather weird name. To set the name of the file, you need to first create the name of the file in Google sheets, and then grab the link to that file and add it as the value for the `ss` argument in `write_sheet`.

```r
write_sheet(sheep_numbers, ss="https://docs.google.com/spreadsheets/d/1cycmCLHvaVuXCebhlbkiVcHylcEIX09RIMJ2o6wpsGc/edit#gid=0")
```

If you refresh your Google sheet, you'll see a new tab named "sheep_numbers". You can manually get rid of the blank tab named "Sheet1".


### Generating a line plot

Now that we have the number of sheep sold each week in the `sheep_numbers` data frame, let's plot the data to see how the sales have varied over the past four years. We will generate the plot using the `ggplot2` package. This is a large package that we'll only start to use in this Code Club. To build a plot with `ggplot2`, you need to tell the `ggplot` function, what your data frame is and what columns in that data frame map to various aspects of the plot. The "mapping" of data to aspects of the plot is done with the `mapping` argument, which takes as a value the output from the `aes` function, which takes as its arguments the various "**aes** thetics" that we want to manipulate.

```r
ggplot(sheep_numbers, aes(x=Date, y=Total))
```

This opens a plotting window with "Date" on the x-axis, "Total" on the y-axis, and a gray background with white grid lines. To visualize those data, we need to add a "geom". There are many geoms in ggplot2, but we'll start with `geom_line`. One of the cool things about `ggplot2` is that we *add* layers to a plot by using the `+` sign.

```r
ggplot(sheep_numbers, aes(x=Date, y=Total)) +
	geom_line()
```

This gives us our line plot - very cool! You'll notice that we got a warning message, "Removed 30 row(s) containing missing values (geom_path)." That's because there were 30 rows with `NA` in the `Total` column. We can move on. To further illustrate the adding nature of `ggplot2`, let's add a theme (check out the earlier Code Club on [`ggplot2`'s themes](/code_club/2020-05-07-fun-with-themes)!).

```r
ggplot(sheep_numbers, aes(x=Date, y=Total)) +
	geom_line() +
	theme_classic()
```

Beyond changing the theme, there are attributes of the line that we can change to give the figure a different look. Reading the output of `?geom_line`, you will find a section titled, "Aesthetics". That section lists the different aesthetics you can modify for the line. These include `color`, `size`, `linetype`, `alpha`, and `group`. We'll briefly cover the `color` and `size` aesthetics in this Code Club, but I encourage you to play with the others to see if you can figure out what they do.

The `color` argument takes either one of R's named colors (see the output of `colors()`) or a hexidecimal string corresponding to the RGB code for a color. We can give the `color` argument to the `geom_line` function.

```r
ggplot(sheep_numbers, aes(x=Date, y=Total)) +
	geom_line(color="red") +
	theme_classic()
```

or

```r
ggplot(sheep_numbers, aes(x=Date, y=Total)) +
	geom_line(color="#236790") +
	theme_classic()
```

The `size` argument takes a number and that number is used to scale the thickness of the line. The default size is 0.5.

```r
ggplot(sheep_numbers, aes(x=Date, y=Total)) +
	geom_line(size=1) +
	theme_classic()
```

We can combine these values to change multiple aesthetics

```r
ggplot(sheep_numbers, aes(x=Date, y=Total)) +
	geom_line(size=1, color="dodgerblue") +
	theme_classic()
```


### Running a smoothed line through line plot

The data are pretty noisy! It kind of looks like the numbers increased between 2016 and 2017, but have been falling off since then. We can fit a smoothed line through the data using `geom_smooth`

```r
ggplot(sheep_numbers, aes(x=Date, y=Total)) +
	geom_line(size=1, color="dodgerblue") +
	geom_smooth() +
	theme_classic()
```

The output tells us that it fit a loess function through the data. With this smoothing model, it looks like the numbers were pretty steady and then started falling after 2018. We can use a different  contrasting color by giving `geom_smooth` a `color` argument

```r
ggplot(sheep_numbers, aes(x=Date, y=Total)) +
	geom_line(size=1, color="dodgerblue") +
	geom_smooth(color="red") +
	theme_classic()
```

I can make the line for the weekly data thinner and gray and the smoothed line thicker to help it stand out more.

```r
ggplot(sheep_numbers, aes(x=Date, y=Total)) +
	geom_line(size=0.25, color="gray") +
	geom_smooth(color="red", size=2) +
	theme_classic()
```

Finally, we can use the `span` argument to tell `geom_smooth` how smooth to make the line. Small values (e.g. 0.1) will make the line squiggly, larger values will make it smoother (e.g. 2).

```r
ggplot(sheep_numbers, aes(x=Date, y=Total)) +
	geom_line(size=0.25, color="gray") +
	geom_smooth(color="red", size=2, span=1) +
	theme_classic()
```

Of course, the figure could use a title to help someone unfamiliar with the data more context about where the data came from and what we're trying to show. But we've done a lot! One thing I don't really like about the figure is the gray confidence band around the smoothed line. Exercise 2 will have you remove that band for me.


## Assignment

1\. In previous Code Clubs, we used this code chunk to download and lightly process the historic weather data from Ann Arbor, MI. Write the command to write the data frame to your own Google drive.

```r
library(lubridate)

aa_weather <- read_csv("https://github.com/riffomonas/generalR_data/blob/master/noaa/USC00200230.csv?raw=true",
	col_type=cols(TOBS = col_double())) %>%
	select(DATE, TMAX, TMIN, TOBS) %>%
	mutate(year = year(DATE),
		month=month(DATE),
		day = day(DATE)) %>%
	filter(year == 1976)
```

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
Go into Google sheets and create a new file called "aa_weather". Grab the link and use it as the value for the `ss` argument

```r
sheet_write(weather, ss=<really long url>)
```
</div>


2\. Look at the help page for `geom_smooth` and see if you can figure out how to get rid of the confidence interval around the curve and how you can make the curve a different color.

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
The argument and value you want to give `geom_smooth` is `se=FALSE`

```r
ggplot(sheep_numbers, aes(x=Date, y=Total)) +
	geom_line(size=0.25, color="gray") +
	geom_smooth(color="red", size=2, span=1, se=FALSE) +
	theme_classic()
```
</div>


3\. Find and plot the total number of lambs sold in each year that I have data. **Hint:** you can use the `year` function from the `lubridate` package to get the year from a date

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
This exercise requires you to remember how to use the `mutate`, `group_by`, and `summarize` functions that we covered in earlier Code Clubs in addition to the `year` function.

```r
library(lubridate)

sheep_numbers %>%
	mutate(year = year(Date)) %>%
	group_by(year) %>%
	summarize(annual_total = sum(Total, na.rm=T)) %>%
	filter(year != 2015 & year != 2020) %>% #these are partial years
	ggplot(aes(x=year, y=annual_total)) +
		geom_line()
```
</div>


4\. I also keep records of my sheep pedigrees, weights, health status, and other information in a [Google sheet](https://docs.google.com/spreadsheets/d/1TVl_h7oUZ-J0Q5LTSz2dWkAxPQ70su2bDY2MEWkXbSg/edit?usp=sharing). In the "weights" tab are the weights of my sheep in pounds over their lives. My longest serving ewe has the eartag 1211. Can you plot her weight over time?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
This exercise requires you to remember how to use the `filter` function that we covered in earlier Code Clubs.

```r
read_sheet("https://docs.google.com/spreadsheets/d/1TVl_h7oUZ-J0Q5LTSz2dWkAxPQ70su2bDY2MEWkXbSg/edit?usp=sharing", sheet="weights", na="NA", col_types="cDcd") %>%
	filter(eartag_n == "1211") %>%
	ggplot(aes(x=date, y=weight)) +
		geom_line() +
		theme_classic()
```
</div>
