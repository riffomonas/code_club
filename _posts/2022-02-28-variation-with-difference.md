---
layout: post
title: "Differences in sampling effort impact Bray-Curtis distances and rarefaction can minimize it (CC191)"
blurb: "Using cut_width to discretize data"
author: "PD Schloss"
date: 2022-02-28 11:00
comments: true
youtube: 6TjzjClQsOg
start_hash: 1738ceb3cb345b9545bdcb13b83139e1553cfd4d
end_hash: f447e005160146f97210a4a05644566ded7cd879
---

Bray-Curtis distances are impacted by the difference the number of observations for each sample being compared. Here Pat shows that the average distance is constant when using rarefaction whereas it is more variable with not rarefying, using relative abundance, or normalization. He shows how to do this analysis using ggplot2's cut_width function. He'll also show how to generate a density plot of the distributions and use geom_smooth to get an overall sense of the variation in the data. He'll show how to synthesize and visualize the data using tools from dplyr, ggplot2, and other tidyverse packages.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/distances/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/distances/tree/{{ page.end_hash }})
