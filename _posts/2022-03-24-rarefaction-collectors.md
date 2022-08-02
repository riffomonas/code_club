---
layout: post
title: "Generating a rarefaction curve from collector's curves in R within the tidyverse (CC198)"
blurb: "Species abundance curves and rarefaction"
author: "PD Schloss"
date: 2022-03-24 11:00
comments: false
youtube: ywHVb0Q-qsM
start_hash: d37cf51d76f2821e413271b12e6cb411eaf87037
end_hash: 6ba9001e14a47f915d4a99339812f7c463c69c9f
---

There's really no reason to generate collector's curves (aka species accumulation curves) or a rarefaction curve. But... seeing how to generate these curves is a convienent approach for illustrating the concepts behind rarefaction. Plus it gives us more practice to learn fun functions from the tidyverse. Along the way we'll look at the effect of the number of iterations and compare this empirical approach to the exact solution that we used in the last episode. I'll show how to synthesize and visualize the data using tools from base R, dplyr, ggplot2 and other tidyverse packages.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/distances/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/distances/tree/{{ page.end_hash }})
