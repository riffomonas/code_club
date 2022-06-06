---
layout: post
title: "How to recreate climate temperature spirals in R with ggplot2 (CC218)"
blurb: "Polar coordinates!"
author: "PD Schloss"
date: 2022-06-02 11:00
comments: true
youtube: NYF9ySYSvwQ
start_hash: 9131b6294f0fe27654777ce3a58f816562356ed9
end_hash: fc62e9ea121994641d0fee1c7369768631dadbda
---

Pat uses tools from the ggplot2 R packge to recreate climate spirals of the annual temperature anomalies by month using NASA's GISS data. This figure shows the deviation in annual global mean temperatures from the normalized temperatures of 1951 to 1980 as a line plot. The lines are colored according to the year. He uses `coord_polar`, `geom_line`, `scale_color_viridis_c`, and a smattering of arguments from the `theme` function to create this provocative visual. All of this is done in R with the help of RStudio.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.end_hash }})
