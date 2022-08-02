---
layout: post
title: "Using the vegan R package to ecological distances (CC188)"
blurb: "Using vegdist and avgdist"
author: "PD Schloss"
date: 2022-02-17 11:00
comments: false
youtube: xyufizOpc5I
start_hash: 3de7aa9efe03683095dfe106d90273ba7d98241d
end_hash: 9b1b931f6b3d310a312b26a1eafb924b5e5eaab1
---

The vegan R package has a powerful set of functions for calcuating the ecological distance between communities. In this episode, Pat shares how to get your data in the right format to use vegdist and avgdist prior to analyzing the distances using NMDS. He discusses using rarefaction with avgdist to control for uneven sampling effort since the Bray-Curtis dissimilarity index is sensitive to uneven sampling effort. We'll use the metaMDS function from vegan and tools from ggplot2 and the tidyverse packages.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/distances/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/distances/tree/{{ page.end_hash }})
