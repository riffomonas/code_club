---
layout: post
title: "Creating a labeled scatter plot in R with ggplot2 (CC157)"
blurb: "Labelled scatter chart"
author: "PD Schloss"
date: 2021-10-25 11:00
comments: true
youtube: VZ4wclK1eCQ
start_hash: 32a63b2b58838805895f5e93a59466ea149705ba
end_hash: e52b5e17cc0599ba02bebd259396494da4cd3032
---

A labeled scatter plot is an effective approach when you want to highlight something about data that you are measuring with two continuous variables. In this episode of Code Club, Pat shows how he would convert a slope chart into a labelled scatter plot to display the change in people's intention to receive the COVID-19 vaccine with data from 2020. He uses `geom_point` from `ggplot2` to create the scatter plot and then shows how to add labels using `geom_label` or `geom_label_repel` (from `ggrepel`). Finally, he also uses `geom_abline`, `coord_fixed`, and `geom_text` to create a legend.  The data depict the percentage of people in 15 countries who would be willing to receive the COVID-19 vaccine as of August and October of 2020.


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
