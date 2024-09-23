---
layout: post
title: "The effect of debug mode when benchmarking R package performance (CC302)"
blurb: "Putting it all together"
author: "PD Schloss"
date: 2024-09-16 9:00
comments: false
youtube: lcVBhgr08O8
start_hash: 0832fcff460056bb795c272f7e36756128f0b3d1
end_hash: 6790bba2b5043fe26b9e36bf178ff1432cdcc9d7
---

When benchmarking the performance of C++ code in R packages, it is important to recognize that the code is compiled in debug mode. This mode is slower than what your user will experience when they install the package on their own computer. In this episode, Pat shows two ways of switching between debug and release mode. He uses the mark function from the {bench} package to test the speed and memory usage of his code written with the help of the {Rcpp} package. This episode is part of an ongoing effort to develop an R package that implements the naive Bayesian classifier for classifying 16S rRNA gene sequences.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

You can browse the state of the repositories at the

* [beginning of the episode](https://github.com/riffomonas/phylotypr/tree/{{page.start_hash}})
* [end of the episode](https://github.com/riffomonas/phylotypr/tree/{{page.end_hash}})
