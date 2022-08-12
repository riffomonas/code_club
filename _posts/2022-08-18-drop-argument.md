---
layout: post
title: "Using the drop argument with factors in count and group_by to include missing data (CC240)"
blurb: "Fun with factors!"
author: "PD Schloss"
date: 2022-08-18 11:00
comments: false
youtube: q9Xq2fliEec
start_hash: 4a631e9ffd5db0da29bc65c768aade0dc6e36bcf
end_hash: af651d1183d9dd51de384ac1c66fd912c186aa29
---

You want to know about the drop argument for `count` and `group_by`, if you have categories that you want to be sure to include in your analysis, but that might be missing. Pat will demonstrate how to use `.drop` with a toy dataset and snow data for the past 130 years that he obtained from local weather data downloaded from NOAA. He'll also show how to convert a factor into a continuous variable for plotting on the x-axis. All this will be done in RStudio with a lot of help from the tidyverse


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.end_hash }})
