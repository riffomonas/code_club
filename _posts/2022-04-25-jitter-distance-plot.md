---
layout: post
title: "Alternatives to ordination in R: Visualizing community change relative to a specific point (CC207)"
blurb: "Replacing ordinations"
author: "PD Schloss"
date: 2022-04-25 11:00
comments: true
youtube: jLVKJ_n6Qd0
start_hash: 8fb8c79fc85ef82ffe890134e9b2d723ebf6e54f
end_hash: 347ad108266734d6ef2de28345b1e75942fc7e68
---

An ordination has a limited set of uses. But are there alternatives to ordination for displaying beta-diversity data? In this episode, Pat shares one method for showing differences between groups of observations using a jittered plot with summary statistics overlayed on the points. He'll demonstrate this approach by using functions from `vegan`, `dplyr`, and `ggplot2`.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/distances/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/distances/tree/{{ page.end_hash }})
