---
layout: post
title: "How I adapt one figure to look like another in R (CC220)"
blurb: "Creating NASA's version of a climate spiral"
author: "PD Schloss"
date: 2022-06-09 11:00
comments: false
youtube: YkJFaQVrD9Y
start_hash: 296dc7c5ceade51d34285ded55faa3c28060f079
end_hash: 57916db41f0392780af3e27a44985093c6e5675c
---

In this episode Pat will use R to adapt a figure made by one creator to look like one made by another creator. He'll share his thought process for thinking about what code needs to be adapted to achieve the desired effect. This will all be done using animated climate spirals of monthly temperature anomalies by month using NASA's GISS data using tools from the ggplot2 and gganimate R packges. This figure shows the deviation in annual global mean temperatures from the normalized temperatures of 1951 to 1980 as a line plot. The lines are colored according to the size of the temperature anomaly. He uses `transition_reveal`, `transition_manual`, `coord_polar`, `geom_line`, `scale_color_viridis_c`, and a smattering of arguments from the theme function to create this provocative visual. All of this is done in R with the help of RStudio.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.end_hash }})
