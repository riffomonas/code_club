---
layout: post
title: "Getting and giving help in a R discussion forum: How to alter borders on plotting symbols (CC167)"
blurb: "Overriding the appearance of ggplot2 legends"
author: "PD Schloss"
date: 2021-11-29 11:00
comments: false
youtube: YkjjQUSutFg
---

Even the most proficient programmers in R need to seek out help from time to time. Discussion forums like those hosted by RStudio and StackOverflow are very helpful, if you know how to ask your questions well. Of course, someone needs to come along and answer your R questions! In this episode of Code Club, Pat shows what makes for a good question and how he goes about answering questions. The specific question in the forum post he looks at asks how to add or remove borders from plotting symbols in R. Along the way, he'll show how to use plotting symbols numbered 21 through 25 and how to override the legend settings for scale_fill_manual, scale_shape_manual, scale_color_manual, and scale_size_manual using guide and override.aes.

In this episode, Pat uses a variety of functions from ggplot2 and resources provided by the people at Rstudio. This episode presents a solution that Pat came up with for a [discussion thread on RStudio Community](https://community.rstudio.com/t/black-border-around-certain-points-in-ggplot/122409).

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

Here's the code that Pat developed in this episode

```R
library(tidyverse)

ggplot(mtcars, aes(x=mpg, y=cyl,
                   fill=as.character(carb),
                   shape=as.character(gear),
                   size=as.character(gear),
                   color=as.character(gear))) +
  geom_point() +
  scale_shape_manual(breaks=c(3, 4, 5),
                     values=c(21, 24, 22),
                     guide=guide_legend(override.aes = list(fill="black",
                                                            color="black"))) +
  scale_color_manual(breaks=c(3, 4, 5),
                     values=c("black", "black", rgb(0, 0, 0, alpha=0))) +
  scale_size_manual(breaks=c(3, 4, 5),
                    values=c(3, 3, 4)) +
  scale_fill_manual(breaks=c(1, 2, 3, 4, 6, 8),
                    values=rainbow(6),
                    guide=guide_legend(override.aes = list(color=rainbow(6),
                                                           shape=18,
                                                           size=5)))
```
