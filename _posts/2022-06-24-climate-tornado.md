---
layout: post
title: "Demonstrating climate change using the ggplot2 R package to create a tornado plot (CC224)"
blurb: "Fun with geom_segment"
author: "PD Schloss"
date: 2022-06-23 11:00
comments: true
youtube: Yebe0IcBFh0
start_hash: 0ab91faa231f6b5dbc08335b61713a5ab80b4a30
end_hash: 63a3d3a4d0f6ab72dfb6d77e8465e36eeb9bb661
---

Pat creates a 'tornado plot' depicting climate change using the ggplot2 R package by plotting the temperature anomaly for April and October by year. The data depicted are monthly temperature anomalies by month using NASA's GISS data using tools from the ggplot2 R package. This figure shows the deviation in annual global mean temperatures from the normalized temperatures of 1951 to 1980 as a series of line segments created using ggplot2's geom_segment function. The lines are colored according to the size of the temperature anomaly. All of this is done in R with the help of RStudio.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.end_hash }})
