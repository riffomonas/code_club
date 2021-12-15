---
layout: post
title: "Applying concepts from Storytelling with Data in R using ggplot2 (CC155)"
blurb: "Using color to highlight message"
author: "PD Schloss"
date: 2021-10-18 11:00
comments: true
youtube: rkQREY_FfqI
start_hash: 7e65f402e6f21fb26cacaa71da87dab423eed609
end_hash: 42e7086e9ff80f61b7a26afd26a7417f212579a2
---

In this episode, I use R's ggplot2 packge to apply some of the concepts from the book, "[Storytelling with Data](https://amzn.to/3mLjpFd)", by Cole Nussbaumer Knaflic's to a slope chart generated using figure showing attitudes towards receiving the COVID-19 vaccine from 2020. One strategy is turning everything gray and then adding color to highlight what is most important to the data storytelling. The data depict the percentage of people in 15 countries who would be willing to receive the COVID-19 vaccine as of August and October of 2020.

Storytelling with Data: https://amzn.to/3mLjpFd (I get a very small comission if you purchase)

In this episode, Pat uses `ggplot2`, `scale_color_manual`, `scale_size_manual`, and manipulating the appearance of figure elements using the `theme` function.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/vaccination_attitudes/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/vaccination_attitudes/tree/{{ page.end_hash }})


### Data

The `august_october_2020.csv` data is available in the [GitHub repository](https://raw.githubusercontent.com/riffomonas/vaccination_attitudes/3f39b9e09618144874ced760c9a6332498e3a19c/august_october_2020.csv).

```
X.1,Total Agree - August 2020,Total Agree - October 2020
Total,77,73
India,87,87
China,97,85
South Korea,84,83
Brazil,88,81
Australia,88,79
United Kingdom,85,79
Mexico,75,78
Canada,76,76
Germany,67,69
Japan,75,69
South Africa,64,68
Italy,67,65
Spain,72,64
United States,67,64
France,59,54
```
