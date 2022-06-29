---
layout: post
title: "How to create a ridgeline plot in R with ggridges in RStudio (CC226)"
blurb: "Showing a change in distributions with time"
author: "PD Schloss"
date: 2022-06-27 11:00
comments: true
youtube: kU4O1LXdz2Y
start_hash: 26d27723b794231b43bd847559c4f0a2df10ea39
end_hash: 4e82b2834d2815a7d82aa467a265ea9ad1a64c26
---

Pat creates a ridgeline plot (aka a joy plot) using the ggridges R package to show the distribution of annual average temperature anomalies. As part of this episode he shows how to read data from a NetCDF file into R and flatten it into a tidy data frame using the data.table R package. He also makes use of tools from the dplyr, ggplot2, and other packages from the tidyverse. The data depicted are the average annual temperature anomalies by latitude and longitude using NASA's GISS data. All of this is done in R with the help of RStudio.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.end_hash }})
