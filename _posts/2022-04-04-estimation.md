---
layout: post
title: "Is richness estimation an alternative to rarefaction? Trying breakaway and Chao1 (CC201)"
blurb: "Integrating complex data with nest/map/unnest"
author: "PD Schloss"
date: 2022-04-04 11:00
comments: true
youtube: xwpMNRt57Zo
start_hash: 9089230a0e61e3d9ce406e06306c21d231cb1d55
end_hash: 767fe1fcd736833a06a4f5119ef199a4839eedbf
---

Richness estimation has been proposed as an alternative to rarefaction because the estimate should be independent of sampling effort. But is it? In this episoe of Code Club, I use the breakaway package in the context of nest, mutate, map, and unnest to assess the sensitivity of richness estimation to sampling effort. I'll show how to get data into the right format to run these functions and how to get the output into the tidyverse. You can find my blog post for this episode at https://www.riffomonas.org/code_club/2022-04-04-estimation. The data were generated in our Kozich et al. 2013 paper (http://doi.org/10.1128/AEM.01043-13) using samples from the Schloss et al. 2012 paper (http://doi.org/10.4161/gmic.21008).


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/distances/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/distances/tree/{{ page.end_hash }})
