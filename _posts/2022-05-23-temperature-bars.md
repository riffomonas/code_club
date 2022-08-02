---
layout: post
title: "Plotting the global temperature index as bars using ggplot2 and NASA GISS data (CC215)"
blurb: "Fun with the scale_fill_gradient functions"
author: "PD Schloss"
date: 2022-05-23 11:00
comments: false
youtube: HDXY0idH7KY
start_hash: da847b1356e3b054fd93af7753b3df518ae6fa54
end_hash: a4feec9e261d17c2d0fb2b00f5a774e5e3d0d11e
---

In this Code Club, Pat uses tools from the ggplot2 R packge to recreate a temperature index plot as bars using NASA's GISS data. This figure shows the deviation in annual global mean temperatures from the normalized temperatures of 1951 to 1980. The bars are colored according to the deviation from the average global temperature. He uses `geom_col`, `scale_fill_gradient`, `scale_fill_gradient2`, `scale_fill_gradientn`, `scale_fillstepsn`, and a smattering of arguments from the `theme` function. All of this is done in R with the help of RStudio.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.end_hash }})
