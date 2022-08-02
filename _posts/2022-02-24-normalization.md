---
layout: post
title: "Is normalization an acceptable alternative to rarefaction? Nope. (CC190)"
blurb: "Rarefaction vs. normalization"
author: "PD Schloss"
date: 2022-02-24 11:00
comments: false
youtube: t5qXPIS-ECU
start_hash: 0f64c79deabf30603205ffc983abc6674cd121b4
end_hash: 1738ceb3cb345b9545bdcb13b83139e1553cfd4d
---

Given the controversy around rarefaction an alternative that has been proposed is to use relative abundance data or normalization. In this Code Club, Pat uses R to compare pairwise distances from a null community model calculated using rarefaction, relative abundances, and normalization using the SRS algorithm with vegdist and avgdist functions from the vegan R package. He'll show how to synthesize and visualize the data using tools from dplyr, ggplot2, and other tidyverse packages.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/distances/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/distances/tree/{{ page.end_hash }})
