---
layout: post
title: "Creating a raster map of global climate change in R with ggplot2's geom_raster (CC227)"
blurb: "Combining <code>geom_raster</code> and <code>facet_grid</code>"
author: "PD Schloss"
date: 2022-07-05 11:00
comments: false
youtube: W1QH83twaak
start_hash: 4e82b2834d2815a7d82aa467a265ea9ad1a64c26
end_hash: a884e4662c2116b308fe54effd3fb2cd0a3989a3
---

Pat uses R to create a faceted raster map displaying the annual global temperature anomalies between 1950 and 2021 using functions from ggplot2. Beyond introducing the `geom_raster` function, Pat will also compare `facet_wrap` and `facet_grid` and demonstrate how to format the appearance of the facet labels. The result is a very cool visualization of the change in temperature across the world for the past 70+ years. He also makes use of tools from the dplyr, ggplot2, and other packages from the tidyverse. The data depicted are the average annual temperature anomalies by latitude and longitude using NASA's GISS data. All of this is done in R with the help of RStudio.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.end_hash }})
