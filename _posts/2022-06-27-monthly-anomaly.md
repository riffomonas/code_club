---
layout: post
title: "Programming a line plot in R to show climate change with and without animation (CC225)"
blurb: "Static and animated line plot"
author: "PD Schloss"
date: 2022-06-27 11:00
comments: false
youtube: DrNQMaIVEVo
start_hash: 63a3d3a4d0f6ab72dfb6d77e8465e36eeb9bb661
end_hash: 26d27723b794231b43bd847559c4f0a2df10ea39
---

Pat shares how he would use r programming to recreate a line plot displaying climate change by month and year using ggplot2. The data depicted are monthly temperature anomalies by month using NASA's GISS data using tools from the ggplot2 R package. This figure shows the deviation in annual global mean temperatures from the normalized temperatures of 1980 to 2015 as a series of lines created using ggplot2's `geom_line` function. The lines are colored according to the average temperature anomaly for the year with `scale_color_gradient2`. Finally, he shows how he would animate the figure with tools from `gganimate`. All of this is done in R with the help of RStudio.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.end_hash }})
