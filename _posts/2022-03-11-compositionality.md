---
layout: post
title: "How to calculate the Aitchison distance in R using two center logratio transformations (CC194)"
blurb: "Aitchison distance is not scale invariant"
author: "PD Schloss"
date: 2022-03-11 11:00
comments: false
youtube: ulo7WatBEAo
start_hash: 7e15b4cd553e07f5cba6c8d6abcd0216166f320f
end_hash: f4f831e1565cd94746a6e587b7f89323c5668ffd
---

The Aitchison distance is the Euclidean distance calculated on species counts subjected to a center logratio transformation (clr). There are several papers that claim whether this distance is invariant or insensitive to differences in sampling depth. Is that true? In this episode, Pat tests Aitchison distances calculated using the robust clr as well as the traditional clr calculated on a count matrix where zero values are imputed using functions from the zComparisons package. He'll compare it to non-rarefied, non-transformed Euclidean distances and to rarefied, non-transformed Euclidean distances. Which approach do you think is most sensitive to uneven sampling effort? He'll show how to synthesize and visualize the data using tools from vegan, dplyr, ggplot2 and other tidyverse packages.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/distances/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/distances/tree/{{ page.end_hash }})
