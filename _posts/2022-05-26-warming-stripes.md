---
layout: post
title: "Using ggplot2 to recreate iconic warming stripes visualization of climate change (CC216)"
blurb: "It's getting warmer"
author: "PD Schloss"
date: 2022-05-26 11:00
comments: false
youtube: NrEdAtSlTMo
start_hash: a4feec9e261d17c2d0fb2b00f5a774e5e3d0d11e
end_hash: 990ac01a82d19d78b9f79c773bdaadac4249fc09
---

Pat uses tools from the ggplot2 R packge to recreate a the iconic warming stripes visualization using NASA's GISS data. This figure shows the deviation in annual global mean temperatures from the normalized temperatures of 1951 to 1980 as a heatmap. The bars are colored according to the deviation from the average global temperature. He uses `geom_tile`, `scale_fillsteps2`, `scale_fillstepsn`, and a smattering of arguments from the `theme` function to create this provocative visual. All of this is done in R with the help of RStudio.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.end_hash }})
