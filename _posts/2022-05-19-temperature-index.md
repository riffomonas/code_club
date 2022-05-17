---
layout: post
title: "Plotting the global temperature index using ggplot2 and NASA GISS data (CC214)"
blurb: "Spoiler: it's getting warmer"
author: "PD Schloss"
date: 2022-05-19 11:00
comments: true
youtube: fskblEWSeWU
start_hash: a428f64b7db493145bf84ec6f38f8e89da258675
end_hash: da847b1356e3b054fd93af7753b3df518ae6fa54
---

In this Code Club, Pat uses tools from the ggplot2 R packge to recreate NASA's temperature index plot using NASA's GISS data. This figure shows the deviation in annual global mean temperatures from the normalized temperatures of 1951 to 1980. He uses `geom_line`, `geom_point`, `geom_smooth`, and a smattering of arguments from the `theme` function. All of this is done in R with the help of RStudio.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.end_hash }})
