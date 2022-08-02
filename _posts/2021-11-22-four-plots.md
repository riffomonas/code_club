---
layout: post
title: "Visualizing the same data four ways wiht ggplot2: slope, dumbbell, scatter, and dot charts (CC165)"
blurb: "Experimenting with ggplot2"
author: "PD Schloss"
date: 2021-11-22 11:00
comments: false
youtube: vd_XG0yBm9s
start_hash: 292867305beb9e0eeeb37202b659c9b55b2b26f2
end_hash: 9cb53f5fd245cbcca3c038ffa45b1f9cc915d09c
---

ggplot2 is a tremendously versatile package for generating attractive figures in R. In this Code Club, Pat uses ggplot2 to generate four visuals using the same data in ggplot2. He'll show how easy it is to generate a slope plot, dumbbell plot (or barbell plot), scatter plot, and dot chart using ggplot2 and how to spice up those figures. The question he is trying to answer with these plots is whether people's stated intention to receive the COVID-19 vaccine in October 2020 matched whether they actually received the vaccine as of October of 2021. He'll demonstrate how to build these figures generated using data from Ipsos and Our World in Data that describes COVID-19 vaccination intention and rates by country and day.

In this episode, Pat uses `geom_point`, `geom_line`, `arrow`, `geom_text_repel`, and more functions from the `ggplot2` and `dplyr` R packages in Rstudio.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/vaccination_attitudes/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/vaccination_attitudes/tree/{{ page.end_hash }})
