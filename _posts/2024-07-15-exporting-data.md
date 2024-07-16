---
layout: post
title: "Adding built in data to an R package (CC294)"
blurb: "Creating a pre-build dataset"
author: "PD Schloss"
date: 2024-07-15 9:00
comments: false
youtube: 9HeqjeS95-U
start_hash: b361e8a6e1fab5c25d942cd5f54187f45a13b7b0
end_hash: 21daf1747139f5dadcbf6104e1fdd1310e0454e1
---

Including data in a package can be useful way to help demonstrate the utility of your R package as well as a convenient way to give your users what they need to run your functions. In this Code Club, Pat shows how to use the {devtools} tools to create two datasets that will potentially ship with his {phylotypr} R package. Along the way, he'll discuss file compression, documentation, and file sizes. This episode is part of an ongoing effort to develop an R package that implements the naive Bayesian classifier.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

You can browse the state of the repository at the

* [beginning of the episode](https://github.com/riffomonas/phylotypr/tree/{{page.start_hash}})
* [end of the episode](https://github.com/riffomonas/phylotyprr/tree/{{page.end_hash}})
