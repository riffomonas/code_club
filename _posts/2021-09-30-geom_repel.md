---
layout: post
title: "How to prevent labels from overlapping in R with ggplot2 and ggrepel packages (CC150)"
blurb: "Separating text labels"
author: "PD Schloss"
date: 2021-09-30 11:00
comments: false
youtube: qsUb2KjJ7tk
start_hash: 9f367142997c366f7b44703bdfab2a478123b985
end_hash: 4b24162b3fc967ff201ded0fb701ced1867b6618
---

A challenge of adding labels to a plot is how to prevent the labels from overlapping with each other. Thankfully, the ggrepel R package can be used with the ggplot2 package to produce an attractive figure without much trouble. The data depict the percentage of people in 15 countries who would be willing to receive the COVID-19 vaccine as of August and October of 2020.

Pat uses the geom_label_repel and `geom_text_repel` functions from `ggrepel` and the `geom_label` function function from `ggplot2` and a variety of functions from the `dplyr`, `showtext`, and `ggtext` packages in RStudio.

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
