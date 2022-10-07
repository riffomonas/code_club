---
layout: post
title: "How to force Snakemake to rerun a rule and visualize a pipeline (CC257)"
blurb: "Fun with reproducibility"
author: "PD Schloss"
date: 2022-10-17 11:00
comments: false
youtube: J6JIKk2MGs4
start_hash: 933c54314be449dcf9c630772a0228fe2dab73ee
end_hash: ed85dbc798f62a4ccdcc7579da3047ae787202d7
---

In this Code Club, Pat shares how to visualize the structure of a Snakemake pipeline and force Snakemake to rerun a rule. In this application we are rerunning a rule because we need to pull fresh data from the NOAA website. After getting fresh data, Pat refactors the R code that he wrote in the last episode for calculating the z-score of the amount of monthly precipitation relative to the same month over the past century. He does this using `filter`, `group_by`, `mutate`, and `select`. The overall goal of this project is to highlight reproducible research practices using a number of tools. The specific output from this project will be a map-based visual that shows the level of drought across the globe.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

You can browse the state of the repository at the

* [beginning of the episode](https://github.com/riffomonas/drought_index/tree/{{page.start_hash}})
* [end of the episode](https://github.com/riffomonas/drought_index/tree/{{page.end_hash}})
