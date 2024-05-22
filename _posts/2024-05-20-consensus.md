---
layout: post
title: "Finding the consensus classification using anonymous functions (CC284)"
blurb: "Fun with anonymous functions"
author: "PD Schloss"
date: 2024-05-20 11:00
comments: false
youtube: TNl8HtUrxl8
start_hash: 7a5cac20692406321bf982b2046f947daf45a84f
end_hash: 59494feddca6eed9250bd4bd93aac9f22971bd74
---

With 100 classification strings, we now need to find the consensus classification across those bootstrapped replicates. We'll make use of anonymous functions with the base R lapply and sapply functions. Of course we'll also make use of test driven development using the testthat package.This episode is part of an ongoing effort to develop an R package that implements the naive Bayesian classifier.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

You can browse the state of the repository at the

* [beginning of the episode](https://github.com/riffomonas/phylotypr/tree/{{page.start_hash}})
* [end of the episode](https://github.com/riffomonas/phylotyprr/tree/{{page.end_hash}})
