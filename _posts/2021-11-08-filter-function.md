---
layout: post
title: "Using dplyr's filter function to pick rows from a data frame in R (CC161)"
blurb: "Everything you ever wanted to know about filter"
author: "PD Schloss"
date: 2021-11-08 11:00
comments: false
youtube: o8lvheRKSbk
start_hash: afbc294673dd460c2272a6ed41f5cf6c5454e957
end_hash: 959afed939a432c84897f4bc67db8b6d2fdf78c3
---

R's `filter` function allows you to create queries to pick rows from a data frame that match your query. It's a tremendously powerful function that we get from the dplyr R package. In this episode we'll see how we can use it to search through a data frame with more than 100000 rows! You can make simple and more sophisticated queries by following a few simple concepts. We'll also see how you can remove NA values using these concepts and using the special `drop_na` function. I'll demonstrate how to use filter with a massive data frame that we downloaded from Our World in Data that describes COVID-19 vaccination rates by country and day. In this episode, Pat uses `filter` and `drop_na` from the `dplyr` R package in Rstudio.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/vaccination_attitudes/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/vaccination_attitudes/tree/{{ page.end_hash }})
