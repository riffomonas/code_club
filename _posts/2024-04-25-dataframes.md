---
layout: post
title: "The tutorial you need to maximize your use data frames in R (CC277)"
blurb: "Benchmarking R's data frames (part 1)"
author: "PD Schloss"
date: 2024-04-25 11:00
comments: false
youtube: J85Q5fqK2Qc
start_hash: a9d23f50d83d98e3055a503233598b28d7ffb4c8
end_hash: 5c91e180094b4c5ea2bb803c13d0738d05b0825f
---

What is your preferred method for building a data frame in R? Do you know its performance characteristics relative to other methods and the size of the data frame? In this tutorial, Pat compares 18 methods of building data frames including similar structures from the tibble, data.table, and Matrix packages. He uses the microbenchmark package to evaluate their speed for different sized vectors. You'll likely be surprised by the results! This episode is part of an ongoing effort to develop an R package that implements the naive Bayesian classifier.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

You can browse the state of the repository at the

* [beginning of the episode](https://github.com/riffomonas/phylotyper/tree/{{page.start_hash}})
* [end of the episode](https://github.com/riffomonas/phylotyper/tree/{{page.end_hash}})
