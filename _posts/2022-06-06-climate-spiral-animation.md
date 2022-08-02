---
layout: post
title: "Recreating animated climate temperature spirals in R with ggplot2 and gganimate (CC219)"
blurb: "Animated climate spiral"
author: "PD Schloss"
date: 2022-06-06 11:00
comments: false
youtube: AYfjdcylAio
start_hash: fc62e9ea121994641d0fee1c7369768631dadbda
end_hash: 95168349ece3c1993e960767a4b4d6124b4abd4a
---

Pat recreates animated climate spirals of the annual temperature anomalies by month using NASA's GISS data using tools from the ggplot2 and gganimate R packges. This figure shows the deviation in annual global mean temperatures from the normalized temperatures of 1951 to 1980 as a line plot. The lines are colored according to the year. He uses `transition_reveal`, `transition_manual`, `coord_polar`, `geom_rect`, `geom_line`, `scale_color_viridis_c`, and a smattering of arguments from the theme function to create this provocative visual. All of this is done in R with the help of RStudio.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.end_hash }})
