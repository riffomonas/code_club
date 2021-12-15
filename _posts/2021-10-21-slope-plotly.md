---
layout: post
title: "Create an interactive slope chart with the plotly and ggplot2 R packages (CC156)"
blurb: "Making an interactive figure"
author: "PD Schloss"
date: 2021-10-21 11:00
comments: true
youtube: tiIdoXxNAxI
start_hash: 7e65f402e6f21fb26cacaa71da87dab423eed609
end_hash: 12576f984254658733e75030e494de793487673a
---

Interactive charts are a great strategy for engaging your audience to investigate the questions they are interested in. Here, Pat will use the plotly R package to take a slope chart generated with ggplot2 to make an interactive figure. The data depict the percentage of people in 15 countries who would be willing to receive the COVID-19 vaccine as of August and October of 2020.

In this episode, Pat uses `plotly`, `ggplot2`, and `dplyr` to make an interactive slope plot.

<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Montserrat&family=Patua+One&display=swap" rel="stylesheet">


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Figure

<iframe style="border: none;" width="700px" height="800px" src="assets/slope-plotly.html"></iframe>

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
