---
layout: post
title: "Generating and classifying bootstrap replicates with test driven development (CC283)"
blurb: "TDD for the win"
author: "PD Schloss"
date: 2024-05-16 11:00
comments: false
youtube: YioVzJiDrjc
start_hash: 0848c315de2ec25638c2d0484a95fbe83349aa00
end_hash: 7a5cac20692406321bf982b2046f947daf45a84f
---

Now that we have a kmer database, we are ready to classify our sequences using the Naive Bayesian approach. We'll start by refactoring some code that we used to make the database and then we'll generate and classify our bootstrap replicates. Of course, everything will be done using test driven development (TDD). This episode is part of an ongoing effort to develop an R package that implements the naive Bayesian classifier.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

You can browse the state of the repository at the

* [beginning of the episode](https://github.com/riffomonas/phylotypr/tree/{{page.start_hash}})
* [end of the episode](https://github.com/riffomonas/phylotyprr/tree/{{page.end_hash}})
