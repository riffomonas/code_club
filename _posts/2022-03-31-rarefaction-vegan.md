---
layout: post
title: "How to rarefy community data in R with vegan and the tidyverse (CC200)"
blurb: "Using rarefy, rrarefy, drarefy, and rarecurve"
author: "PD Schloss"
date: 2022-03-31 11:00
comments: true
youtube: _OEdFjc1D9I
start_hash: 2a2da903d1216a50d61ade7837d9e60e4310fd24
end_hash: 7945a48b9f404242f3ae3ca28624df31ddf7a422
---

The vegan R package has a lot of useful functions for doing community ecology analysis including rarefaction with the rarefy, rrarefy, drarefy, and rarecurve functions. In this episode, I'll show how to get data into the right format to run these functions and how to get the output into the tidyverse. You can find my blog post for this episode at https://www.riffomonas.org/code_club/2022-03-31-rarefaction-vegan. The data were generated in our Kozich et al. 2013 paper (http://doi.org/10.1128/AEM.01043-13) using samples from the Schloss et al. 2012 paper (http://doi.org/10.4161/gmic.21008).


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/distances/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/distances/tree/{{ page.end_hash }})
