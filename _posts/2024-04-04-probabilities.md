---
layout: post
title: "Using base R and testthat to calculate probabilities (CC271)"
blurb: "testthat and TDD"
author: "PD Schloss"
date: 2024-04-04 11:00
comments: false
youtube: qUkyQjF1k3o
start_hash: 917970118bc2f64e3ecdc4ae03a028265862e8dc
end_hash: 6d206f9d22969a9654ff1eb279343fe2d8740fb6
---

Watch and code along with Pat as he uses test driven development and base R to count kmers and calculate probabilities for a naive Bayesian sequence classifier. Pat generates all possible kmers for a sequence and then all sequences in a collection. These kmer counts are then used to generate the word specific priors and genus-specific conditional probabilities that are needed to train the naive Bayesian classifier for 16S rRNA gene sequences. Along the way, Pat continues to use Test Driven Development using the testthat R package and a number of tools from base R. This episode is part of an ongoing effort to develop an R package that implements the naive Bayesian classifier.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

You can browse the state of the repository at the

* [beginning of the episode](https://github.com/riffomonas/phylotyper/tree/{{page.start_hash}})
* [end of the episode](https://github.com/riffomonas/phylotyper/tree/{{page.end_hash}})
