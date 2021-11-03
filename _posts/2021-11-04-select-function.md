---
layout: post
title: "Using the select function and its helper functions in R to pick columns from a data frame (CC160)"
blurb: "Everything you ever wanted to know about select"
author: "PD Schloss"
date: 2021-11-04 11:00
comments: true
youtube: kj8u2aDO3Go
start_hash: b9341e981e054e5ff4bb54f8a23140c28679f223
end_hash: afbc294673dd460c2272a6ed41f5cf6c5454e957
---

I have a data frame with 65 columns, but only want 4 of those. What am I to do? This is a great place to use the select function and its helper fuctions from the R dplyr package. Pat will show how to use the select function to get columns you want by their name along with the starts_with, ends_with, contains, and matches helper function when you aren't certain of the column names you want. We'll also see how you can use the select function to rename columns. We will practice these tools with a massive data frame that we downloaded from Our World in Data that describes COVID-19 vaccination rates by country and day.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the demo at the
* [beginning of the episode](https://github.com/riffomonas/vaccination_attitudes/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/vaccination_attitudes/tree/{{ page.end_hash }})
