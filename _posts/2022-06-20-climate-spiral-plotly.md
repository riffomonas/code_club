---
layout: post
title: "Using R to make a 3D interactive figure showing climate change with plotly (CC223)"
blurb: "Interactive! 3D!"
author: "PD Schloss"
date: 2022-06-20 11:00
comments: true
youtube: PytBiFU0rEc
start_hash: 9317fb0e5a338c7dc11e2a7ba1aa6b6b04624b08
end_hash: 0ab91faa231f6b5dbc08335b61713a5ab80b4a30
---

Pat will show you how to show evidence of climate change using R to create a 3D interactive climate spiral using the plotly R package. This will all be done using monthly temperature anomalies by month using NASA's GISS data using tools from the `ggplot2` and `ggplotly` R packages. This figure shows the deviation in annual global mean temperatures from the normalized temperatures of 1951 to 1980 as a line plot. The lines are colored according to the size of the temperature anomaly. He uses `ggplotly` and `plot_ly`, and a smattering of arguments from the theme function to create this provocative visual. All of this is done in R with the help of RStudio.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Output

<iframe width="100%" height="800px" src="assets/climate_spiral_plotly.html"></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/climate_viz/tree/{{ page.end_hash }})
