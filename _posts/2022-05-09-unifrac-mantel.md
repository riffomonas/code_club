---
layout: post
title: "Using the mantel test to compare ecological matrices using the vegan R package (CC211)"
blurb: "Comparing phylogenetic and bin-based approaches"
author: "PD Schloss"
date: 2022-05-09 11:00
comments: true
youtube: EXNOgmUyPfY
start_hash: d7ee96626f4868ebf9fd8250006d8b9034b01c42
end_hash: b64edeb08f046ff2962314f5b374a456ad805db3
---

The Mantel test is useful for comparing distances matrices and is straightforward to do with the mantel function from the vegan R package. In this Code Club, Pat will use the Mantel test to compare bin-based and phylogenetic beta diversity matrices (weighted and unweighted UniFrac) and to compare membership and structure based metrics of beta diversity. What correlates with what? He'll answer this question using data generated with mothur and R functions from `vegan`, `dplyr`, and `ggplot2`.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/distances/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/distances/tree/{{ page.end_hash }})
