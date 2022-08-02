---
layout: post
title: "Repeating and parallelizing a function in R with the purrr and furrr packages (CC192)"
blurb: "Relationship between sampling depth and Bray-Curtis distances"
author: "PD Schloss"
date: 2022-03-03 11:00
comments: false
youtube: _-lQZMtHdZg
start_hash: f447e005160146f97210a4a05644566ded7cd879
end_hash: 766198091663507b5b625a32a56a4f24e2102e4f
---

In this episode Pat writes a function in R that needs to be repeated for different input values. He shows how to do this with purrr's map_dfr and then speeds things up by parallelizing it with furrr's future_map_dfr. The specific question in this episode is how the sampling depth of communities impacts pairwise Bray-Curtis distances calculated using avgdist from vegan. He'll show how to synthesize and visualize the data using tools from dplyr, ggplot2, and other tidyverse packages.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/distances/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/distances/tree/{{ page.end_hash }})
