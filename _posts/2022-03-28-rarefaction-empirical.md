---
layout: post
title: "Using R to compare empirical and exact rarefaction values (CC199)"
blurb: "It's all in the number of iterations"
author: "PD Schloss"
date: 2022-03-24 11:00
comments: false
youtube: ywHVb0Q-qsM
start_hash: 6ba9001e14a47f915d4a99339812f7c463c69c9f
end_hash: d088d721987255fdd7d4471527798f35cdd94131
---

In this Code Club, I'll use R to show you how to use an empirical approach to rarefaction with tools from the tidyverse packages. We'll look at the effect of different numbers of iterations on the precision of the rarefied richness values relative to the exact rarefactoin value. You can find my blog post for this episode at https://www.riffomonas.org/code_club/2022-03-28-rarefaction-empirical. The data were generated in our Kozich et al. 2013 paper (http://doi.org/10.1128/AEM.01043-13) using samples from the Schloss et al. 2012 paper (http://doi.org/10.4161/gmic.21008).


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/distances/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/distances/tree/{{ page.end_hash }})
