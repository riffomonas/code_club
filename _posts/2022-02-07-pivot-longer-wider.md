---
layout: post
title: "Reshaping data in R to be long or wide with pivot_longer and pivot_wider (CC185)"
blurb: "Filtering a distance matrix in R"
author: "PD Schloss"
date: 2022-02-07 11:00
comments: true
youtube: ZVddiwrbrr4
start_hash: 6e12a7cf418c61509bbdfea79af830fa25dc369d
end_hash: e14ff12ef551aec7bfd74323551d2726cabccd8d
---

R's tidyverse tidyr package has powerful functions for reshaping data frames to be long or wide using the `pivot_longer` and `pivot_wider` functions, respectively. In this episode, Pat Schloss will show how to use these functions to convert a distance matrix into a format that is easier to use with other tools like `inner_join` and `filter`. The goal of this episode is to filter a square distance matrix to only contain distances from a subset of samples. We'll do all this in RStudio using the tidyverse package.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/distances/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/distances/tree/{{ page.end_hash }})
