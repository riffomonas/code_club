---
layout: post
title: "Finding and loading data into R (CC159)"
blurb: "Loading data without downloading a file"
author: "PD Schloss"
date: 2021-11-01 11:00
comments: true
youtube: iChfSHj7G30
start_hash: e5aa199f9179f8957ef47dffc31aa6859067085d
end_hash: b9341e981e054e5ff4bb54f8a23140c28679f223
---

I would like to compare the percent of people who said they would receive the COVID-19 vaccine in 2020 to those who actually did it in 2021. How would you go about finding and loading that data into R? Check out this episode where I show how I went about finding current vaccination rates and can load the data automatically in R. We'll use `read_csv`/`read_tsv` to load data from a file and directly from a web address or URL.

In this episode, Pat uses `read_csv` and `read_tsv` from the `readr` R package in Rstudio.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/vaccination_attitudes/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/vaccination_attitudes/tree/{{ page.end_hash }})
