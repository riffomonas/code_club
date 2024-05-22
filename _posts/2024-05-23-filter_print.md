---
layout: post
title: "Base R's paste and paste0 functions: how to use the sep and collapse arguments (CC285)"
blurb: "How to use R's paste and paste0 functions"
author: "PD Schloss"
date: 2024-05-23 11:00
comments: false
youtube: 2sZCI5UhFiM
start_hash: 59494feddca6eed9250bd4bd93aac9f22971bd74
end_hash: cba9943928b7b81b793981435b9ccbe89724c17a
---

The paste and paste0 functions are powerful tools for creating strings in base R that don't require dependencies. You can even paste together two vectors element-wise separating the values with the sep argument and then paste together all elements of the resulting vector with the collapse argument. Pat will use paste and other base R functions to filter taxonomy data by confidence score and then output the result in an attractive string using paste. This episode is part of an ongoing effort to develop an R package that implements the naive Bayesian classifier.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

You can browse the state of the repository at the

* [beginning of the episode](https://github.com/riffomonas/phylotypr/tree/{{page.start_hash}})
* [end of the episode](https://github.com/riffomonas/phylotyprr/tree/{{page.end_hash}})
