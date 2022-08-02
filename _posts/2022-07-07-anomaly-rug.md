---
layout: post
title: "A rug chart in R with ggplot2's geom_segment showing latitudinal temperature anomalies (CC228)"
blurb: "Fun with <code>geom_segment</code>"
author: "PD Schloss"
date: 2022-07-07 11:00
comments: false
youtube: UbuKFW5eul8
start_hash: a884e4662c2116b308fe54effd3fb2cd0a3989a3
end_hash: 867ed3dfae24b2c0b53b575ecc8e272272e5069a
---

Pat uses R to create a rug chart using ggplot2's `geom_segment` function and then sylizes the heck out of it to try to match an animated version of the same figure from GISS at NASA. The result is a very cool visualization of annual latitudinal temperature anomalies over the past 140+ years. He also makes use of tools from the dplyr, ggplot2, and other packages from the tidyverse. The data depicted are the average annual temperature anomalies by latitude using NASA's GISS data. All of this is done in R with the help of RStudio.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.end_hash }})
