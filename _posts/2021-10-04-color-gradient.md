---
layout: post
title: "Creating a color gradient in R with ggplot2 (CC151)"
blurb: "Manipulating color with ggplot2"
author: "PD Schloss"
date: 2021-10-04 11:00
comments: true
youtube: FQw1mBFNi8E
start_hash: 4b24162b3fc967ff201ded0fb701ced1867b6618
end_hash: 57327fef3fd668213b94883958b95217111f888e
---

The ggplot2 R package offers a lot of flexibility in representing color using discrete or continuous values as a gradient. In this episode of Code Club, Pat will use `scale_color_manual`, `scale_color_gradient`, and `scale_color_gradient2` to customize the change in attitudes around receiving the COVID-19 vaccine. The data depict the percentage of people in 15 countries who would be willing to receive the COVID-19 vaccine as of August and October of 2020.

Pat uses the `scale_color_manual`, `scale_color_gradient`, and `scale_color_gradient2` functions from `ggplot2` and a variety of functions from the `dplyr`, `showtext`, and `ggtext` packages in RStudio. Nearly all of the concepts Pat talks about for color can also be used with fills using `scale_fill_manual`, `scale_fill_gradient`, and `scale_fill_gradient2`.

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
