---
layout: post
title: "Using dplyr's slice and arrange functions in R to order and pick rows from a data frame (CC162)"
blurb: "How to arrange rows in R"
author: "PD Schloss"
date: 2021-11-11 11:00
comments: false
youtube: tb054Z3AMN4
start_hash: 959afed939a432c84897f4bc67db8b6d2fdf78c3
end_hash: 0bed39115831c015e5158abed17d5867949af933
---

In this episode of Code Club, Pat uses `dplyr`'s slice and arrange functions to show how you can order and grab rows out of a data frame using R. He'll cover how to do an ascending and descending sort using arrange and desc and the `slice_head`, `slice_tail`, `slice_max`, `slice_min`, and `slice_sample` functions to grab rows with particular characteristics. We'll also talk about `top_n`. He'll demonstrate how to use these functions with a massive data frame that we downloaded from Our World in Data that describes COVID-19 vaccination rates by country and day.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/vaccination_attitudes/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/vaccination_attitudes/tree/{{ page.end_hash }})
