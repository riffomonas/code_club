---
layout: post
title: "Brute force building a kmer database in R (CC272)"
blurb: "Putting things together"
author: "PD Schloss"
date: 2024-04-08 11:00
comments: false
youtube: HhjrETgFS70
start_hash: 6d206f9d22969a9654ff1eb279343fe2d8740fb6
end_hash: cdfa2cde961ad1b6e7ef987b90ca52beb0f6526e
---

Pat attempts to put together the R code that he's been writing to create a single function that will build a kmer database. Along the way, Pat continues to use Test Driven Development using the `testthat` R package and a number of tools from base R. After making sure everything works by TDD, he attempts to build the database with the most recent version of the RDP training set. This episode is part of an ongoing effort to develop an R package that implements the naive Bayesian classifier.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

You can browse the state of the repository at the

* [beginning of the episode](https://github.com/riffomonas/phylotyper/tree/{{page.start_hash}})
* [end of the episode](https://github.com/riffomonas/phylotyper/tree/{{page.end_hash}})
