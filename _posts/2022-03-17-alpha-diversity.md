---
layout: post
title: "Using vegan to calculate alpha diversity metrics within the tidyverse in R (CC196)"
blurb: "Writing functions in R"
author: "PD Schloss"
date: 2022-03-17 11:00
comments: false
youtube: wq1SXGQYgCs
start_hash: 64df17d165f28fc283a0f892c90f482650af07e7
end_hash: 48ba60df47e76afb9ae34b275a07dd4d730f3cfa
---

Among the useful tools in the vegan R package are functions for calculating alpha diversity metrics and indices. In this episode of Code Club, Pat shows how to create our own versions of these functions and how we can implement either version in a `group_by` / `summarize` pipeline using dplyr. We'll also take a look at whether there is an effect of sampling effort on the observed richness, shannon diversity, and simpson diversity. He'll show how to synthesize and visualize the data using tools from vegan, dplyr, ggplot2 and other tidyverse packages.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/distances/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/distances/tree/{{ page.end_hash }})
