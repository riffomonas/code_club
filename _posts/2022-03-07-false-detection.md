---
layout: post
title: "Rarefaction controls the false positive rate when using adonis from the vegan R package (CC193)"
blurb: "Creating a simulation in R"
author: "PD Schloss"
date: 2022-03-07 11:00
comments: false
youtube: 5Mry6nXpdfE
start_hash: 766198091663507b5b625a32a56a4f24e2102e4f
end_hash: 249d6d18ce53cbd9fd440ebbfc649e4d61df8fcc
---

Pat creates a simulation to test whether rarefaction can control the false positive rate when using the adonis function from the vegan R package. He reviews alternatives to rarefaction and how to implement them before using vegdist and avgdist from vegan to calculate distances. He then shows how to test for significance using adonis (also called PERMANOVA). Finally, he shows how to run 100 iterations of the test with different random number generator seeds using purrr's map_dfr and then speeds things up by parallelizing it with furrr's future_map_dfr. Rarefaction was the only method able to control the false positive rate with non-rarefied, relative abundance, and normalized data having high false positive rates. He'll show how to synthesize and visualize the data using tools from dplyr and other tidyverse packages.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/distances/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/distances/tree/{{ page.end_hash }})
