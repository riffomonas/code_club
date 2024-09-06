---
layout: post
title: "Using the precommit R package to automate testing code and documentation quality(CC301)"
blurb: "Pre-commit hooks with R package development"
author: "PD Schloss"
date: 2024-09-09 9:00
comments: false
youtube: y1nwosH8ybk
start_hash: 508b51c1fe6748899feec1a178b5f162d2b26c9c
end_hash: 0bd3197724d835f311a40d760fb59370ac40c3fc
---

Building R packages has many small steps that are easy to forget. With pre-commit hooks we can make sure that these steps are implemented before we commit and push our code to GitHub. This is made easier using the precommit R package with the pre-commit utility and GitHub actions. Watch along as Pat learns to use these resources by implementing them on his phylotypr package. This episode is part of an ongoing effort to develop an R package that implements the naive Bayesian classifier for classifying 16S rRNA gene sequences.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

You can browse the state of the repositories at the

* [beginning of the episode](https://github.com/riffomonas/phylotypr/tree/{{page.start_hash}})
* [end of the episode](https://github.com/riffomonas/phylotypr/tree/{{page.end_hash}})
