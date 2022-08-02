---
layout: post
title: "How to use R to create a heatmap from a distance matrix with ggplot2 (CC209)"
blurb: "Building a heatmap from a matrix"
author: "PD Schloss"
date: 2022-05-02 11:00
comments: false
youtube: 3iOTGaWQZp4
start_hash: f57b05f5fca827839d52500122d9db572c90c8d1
end_hash: e1581cd6634edfa6effabd78e4c5c19756f57b4c
---

In this Code Club, Pat shows how to generate a heatmap in R from a distance matrix generated using vegan. He starts with making a heatmap for a Bray-Curtis distance matrix and then incorporates Jaccard distances for the same data. Along the way he provides a critique of this approach to visualizing distances between samples. He'll demonstrate this approach by using functions from `vegan`, `dplyr`, and `ggplot2`.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/distances/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/distances/tree/{{ page.end_hash }})
