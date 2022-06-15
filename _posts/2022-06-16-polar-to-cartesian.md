---
layout: post
title: "How to create spirals in cartesian coordinates with ggplot2 in R (CC222)"
blurb: "Fun trigonometry fun"
author: "PD Schloss"
date: 2022-06-16 11:00
comments: true
youtube: kXbpAhWF6iw
start_hash: 2289fc8229b6fe10ead9bd2f7e88fb0bc07243f6
end_hash: 9317fb0e5a338c7dc11e2a7ba1aa6b6b04624b08
---

In this episode Pat will show you how to create spirals in cartesian coordinates using ggplot2 without the aid of coord_polar. He'll then use this strategy to recreate one of the climate spirals. Get ready for some trigonometry! While coord_polar was a lot easier to use, this will set us up well for developing related data visualizations. This will all be done using animated climate spirals of monthly temperature anomalies by month using NASA's GISS data using tools from the `ggplot2` and `gganimate` R packages. This figure shows the deviation in annual global mean temperatures from the normalized temperatures of 1951 to 1980 as a line plot. The lines are colored according to the size of the temperature anomaly. He uses `transition_reveal`, `geom_path`, and a smattering of arguments from the `theme` function to create this provocative visual. All of this is done in R with the help of RStudio.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.end_hash }})
