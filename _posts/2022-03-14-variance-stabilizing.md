---
layout: post
title: "How to use DESeq2's variance stabilizing transformation with microbiome data (CC195)"
blurb: "VST is not scale invariant"
author: "PD Schloss"
date: 2022-03-14 11:00
comments: false
youtube: E9QS4dRqWS8
start_hash: d807b46768ab346bd1a533e88ced075d4a5ddbfb
end_hash: c748a4086e9d1811810385ee2b11e270bbf1825a
---

Performing microbiome analyses using variance stabilizing transformation from DESeq2 has been recommended as an approach to control for uneven sampling effort so that we can avoid using rarefaction in analyses. In this Code Club, Pat will use vst to transform his microbiome data and then assess whether distances calculated from these distances are sensitive to uneven sampling effort. He'll compare these distances to those calculated using rarefaction. Which approach do you think is most sensitive to uneven sampling effort? He'll show how to synthesize and visualize the data using tools from vegan, DESeq2, dplyr, ggplot2 and other tidyverse packages.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/distances/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/distances/tree/{{ page.end_hash }})
