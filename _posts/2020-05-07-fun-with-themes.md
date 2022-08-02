---
layout: post
title: "Fun with ggplot2's themes"
author: "PD Schloss"
date: 2020-05-07
blurb: "Ever feel like <code>ggplot2</code>'s <code>theme</code> function was a black hole? We'll break it down"
comments: false
---

If you're like me, you're not a big fan of `ggplot2`'s default plot style. Somewhere I heard that developers make these types of default themes ugly so that you have to change them. Aside from using some of the other built in themes (e.g. `theme_classic`, `theme_dark`), there are the `ggthemes` package with a bunch of other canned themes and the `theme` function. For today's Code Club we are going to see how you can manipulate the built in themes, pick themes from `ggthemes`, and use the `theme` function to make your own modifications. Altering the styling of a figure can be helpful if you need to modify a figure to adapt something you made for a paper to something you'd like to show in a presentation. Alternatively, as we'll see, if you have a "brand" and want to have a consistent style across figures, you can create a theme that is then used in each case.

We'll take a different approach to previous Code Clubs in the video for this week. I'll present a brief lesson and then turn you loose on your own to work through some lessons. Finally, I'll come back and give you my solutions. Don't watch the video straight through without firing up RStudio and trying the code and exercises yourself! Please be sure to see the [setup instructions](/code_club/setup-instructions) before you get going.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/CHApFmFoP_o" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Prompt

Here is a code chunk that was modified from the second exercise in the [first Code Club](2020-03-26-candy-crush). You'll notice I removed the `theme_classic()` line from my ggplot block and that I assigned the block to the variable `p`.

```r
library(tidyverse)

candy_data <- read_csv("https://raw.githubusercontent.com/fivethirtyeight/data/master/candy-power-ranking/candy-data.csv",
	col_types="clllllllllddd")


pluribus_data <- candy_data %>%
	filter(pluribus & (chocolate | fruity)) %>%
	pivot_longer(cols=c(chocolate, fruity), names_to="type", values_to="answer") %>%
	filter(answer)

p <- pluribus_data %>%
	ggplot(aes(x=type, y=winpercent, color=type)) +
	geom_jitter(width=0.1) +
	scale_x_discrete(breaks=c("chocolate", "fruity"), labels=c("Chocolate", "Fruity")) +
	coord_cartesian(ylim=c(0,100)) +
	labs(y="Contests won (%)",
		x="Type of candy",
		title="Bite-sized candies containing chocolate are preferred to candy without",
		subtitle="Data collected by FiveThirtyEight.com")
```

That `p` variable holds the "value" of the plot. If you enter "p" at the R prompt it will generate the figure. This is often done so that you don't have to repeat all the lines of code to that point and you can add additional layers to the plot. For example, this line of code applies `theme_bw` to our figure

```r
p + theme_bw()
```

### Built in themes

`theme_bw` is one of the themes that is built into `ggplot2`. Here are some of the others that you can use...

```r
p + theme_gray() # the default
p + theme_linedraw()
p + theme_light()
p + theme_dark()
p + theme_minimal()
p + theme_classic()
p + theme_void()
```


### Extra themes

If you want more options, you can check out the `ggthemes` package, which has people's attempts at trying to recreate the themes they see from other software packages (e.g. `theme_excel`) and websites (e.g. `theme_economist`). You can find a gallery of themes in `ggthemes` and in other packages (e.g. the `xkcd` package) at [allYourFigureAreBelongToUs](https://yutannihilation.github.io/allYourFigureAreBelongToUs/). Be sure to install `ggthemes` or the package with the theme you want before attempting to use the theme

```r
#install.packages("ggthemes")
library(ggthemes)

p + theme_economist() #
p + theme_excel()
p + theme_fivethirtyeight()
```

### Passing arguments into `theme_xxxxx` functions

You can manipulate a lot in a figure by giving these default themes a few different arguments including `base_size`, `base_family`, `base_line_size`, and `base_rect_size`. These will change the font size, font type, thickness of the lines, and the size of border on rectangular elements.

```R
p + theme_classic()
p + theme_classic(base_size = 20) #a number, default = 11
p + theme_classic(base_family = "mono") #a font family, default = sans (serif, mono, symbol); see extrafont package
p + theme_classic(base_line_size = 2) #a number, size for line elements, default = base_size/22
```

You should notice that when you change something like `base_size` all of the font sizes for the titles, axis titles, axis labels and axis values are scaled according to this base value. We can say that the value is inherited as we go down the cascade of related styles.


### Digging into `theme`

These options may only get you so far. You might find that you like `theme_classic`, but want to change one or two things. This brings us to the `theme` function. This function has one of the more intimidating help pages

```r
?theme
```

I'd encourage you to skip the `Usage` section and go to the `Arguments` section for an easier view of the various things you can manipulate. You'll notice that there is generally a hierarchy to how they're named. Unfortunately, they are mainly listed alphabetically not hierarchically
* the first word indicates what gets manipulated (e.g. plot, legend, etc)
* the second word indicates what about the first word gets manipulated (e.g. plot.background)
* the third word indicateds a finer level of specificity (e.g. legend.box.background)
* sometimes the third word overwrites what happens with the second word (e.g. axis.title vs axis.title.x)

When setting the values for the theme parameters for some arguments they take a character or logical value. More frequently they take one of five functions.

* `margin()`
* `element_blank()`
* `element_rect()`
* `element_line()`
* `element_text()`
* `unit()` see `?grid::unit` for the various units that are available

You should look at the help pages for those functions to get a sense of their syntax. The arguments that these functions take can be a bit opaque
* `fill` / `color` / `colour` ~ colors
* `linetype` is an integer between 0 and 8, name of the line type, or hexidecimal coding
* `linend` can be "round", "butt", or "square"
* `arrow` see `?grid::arrow`
* `face` can be "plain", "italic", "bold", "bold.italic"
* `hjust` is a number between 0 and 1 to indicate the horizontal justification (0 for left, 0.5 for center, and 1 for right justification)
* `vjust` is a number between 0 and 1 to indicate the vertical justification (0 for top, 0.5 for center, and 1 for bottom)
* You can also use `rel()` so set a size **rel** ative to the size of the parent entity (e.g. `rel(1.5)` in `element_text` for `plot.title` would make it 50% larger than the value of `base_size`)

Let's see how we can put this together to make something garish...

```r
p +
	theme_gray() +
	theme(plot.title = element_text(color="red", family="mono", size=18),
		axis.title = element_text(color="blue"),
		axis.title.x = element_text(size = rel(2.0), family="serif"),
		legend.background = element_rect(color="black", fill=NA),
		plot.background = element_rect(fill="lightgray"),
		panel.grid.major.y = element_line(linetype=3),
		panel.background = element_rect("pink"))
```

### Looking behind the curtain of a `theme_xxxx` function
One trick you can use to figure out how a theme is made is to recall that if you enter the name of a function at the prompt without any arguments or the parentheses you will often get the code for the function. Well, `theme_excel` is a function! For the assignments, try not to use this approach to engineer your own version of these `theme_xxxx` functions.


## Assignment
1\. Without using `theme_xxxxxx` functions, make `p` look like `p + theme_classic()` using `theme()`. Hint: Within RStudio's "Plots" panel you can use the arrow buttons to toggle between versions of plots

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```R
p + theme_classic()

p + theme(
	panel.grid = element_blank(),
	panel.background = element_blank(),
	axis.line = element_line(lineend ="round"),
	legend.key = element_blank()
	)
```
</div>


2\. Without using `theme_xxxxxx` functions, make `p` look like `p + theme_dark()` using `theme()`.


<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```r
p + theme_dark()

p + theme(
	line = element_line(size=0.25),
	panel.background = element_rect(fill="gray50"),
	panel.grid = element_line(color="gray40"),
	legend.key = element_rect(fill="gray50", color=NA)
	)
```
</div>



3\. Recreate the `theme_fivethirtyeight` theme using `theme()`.

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```r
p + theme_fivethirtyeight()

p + theme(
	text = element_text(size=12),
	plot.title=element_text(face="bold", size=18),
	plot.margin = unit(c(1, 1, 1, 1), unit="lines"),
	plot.background = element_rect(fill="gray95", color=NA),
	panel.background = element_blank(),
	axis.title = element_blank(),
	panel.grid = element_line(color="gray80"),
	panel.grid.minor = element_blank(),
	axis.ticks = element_blank(),
	legend.position = "bottom",
	legend.background = element_blank(),
	legend.key = element_blank()
)
```
</div>
