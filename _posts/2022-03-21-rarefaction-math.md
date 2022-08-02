---
layout: post
title: "Implementing the mathematical formula for rarefaction to measure richness in R (CC197)"
blurb: "Finding an exact solution for rarefaction"
author: "PD Schloss"
date: 2022-03-21 11:00
comments: false
youtube: 7oH9KBLeqDc
start_hash: 0f8b592929e2d5878ed341ef7111cd3da69306e0
end_hash: d37cf51d76f2821e413271b12e6cb411eaf87037
---

Did you know that there's a mathematical formula to measure richness by rarefaction? It leverages all sorts of concepts that we rarely think about, but are important to appreciate to full understand rarefaction. In this episode we'll use the R factorial, combn, choose, and lchoose functions to get an exact solution for the richness of a community rarefied to a specific sampling depth. Along the way we'll also discuss the power of working with logrithms. I'll show how to synthesize and visualize the data using tools from base R, dplyr, ggplot2 and other tidyverse packages.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/distances/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/distances/tree/{{ page.end_hash }})
