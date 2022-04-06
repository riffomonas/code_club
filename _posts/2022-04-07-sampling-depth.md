---
layout: post
title: "How to find the best sampling depth for rarefaction (CC202)"
blurb: "This isn't an arbitrary process"
author: "PD Schloss"
date: 2022-04-07 11:00
comments: true
youtube: c7H8jLjTxSE
start_hash: 34387e1f203317d5071244728fe5ffe292688a3e
end_hash: 52a70390b74b5d3fd4dd9a214927ec1b1424d64f
---

One critique of rarefaction is that the sampling depth people pick is arbitrary. Is that true? In this Code Club, I'll show you my thought process for picking the best sampling depth to use for rarefying my data by rarefaction. Along the way we'll look at approaches to visualize the number of sequences per sample. We'll also see how to calculate Good's coverage to ascertain how well we have sampled our communities for a desired depth of sequencing. You can find my blog post for this episode at https://www.riffomonas.org/code_club/2022-04-04-estimation. The data were generated in our Kozich et al. 2013 paper (http://doi.org/10.1128/AEM.01043-13) using samples from the Schloss et al. 2012 paper (http://doi.org/10.4161/gmic.21008).


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/distances/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/distances/tree/{{ page.end_hash }})
