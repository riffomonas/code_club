---
layout: post
title: "Using lintr and styler to improve the quality and readability of R code (CC300)"
blurb: "Stylish R code"
author: "PD Schloss"
date: 2024-08-26 9:00
comments: false
youtube: y1nwosH8ybk
start_hash: bc86d58e9755af9bbfd740be92ca9670b6f024e3
end_hash: 508b51c1fe6748899feec1a178b5f162d2b26c9c
---

The tidyverse style guide is a popular set of rules that you can apply to R code to improve the readability and quality of your code. Thankfully, the lintr package is able to screen our code (it's a "linter") to see where we may have violated some of these rules and the styler package is able to correct many of these violations. I show how to use these tools to pass the linter. Then I show how to incorporate the linter into a GitHub action to lint our R code whenever we push code to GitHub. This episode is part of an ongoing effort to develop an R package that implements the naive Bayesian classifier for classifying 16S rRNA gene sequences.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

You can browse the state of the repositories at the

* [beginning of the episode](https://github.com/riffomonas/phylotypr/tree/{{page.start_hash}})
* [end of the episode](https://github.com/riffomonas/phylotypr/tree/{{page.end_hash}})
