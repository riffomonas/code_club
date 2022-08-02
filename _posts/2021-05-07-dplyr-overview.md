---
layout: post
title: "How to clean and join data from mothur with the dplyr R package (CC101)"
blurb: "Using dplyr with mothur data"
author: "PD Schloss"
date: 2021-05-07 11:30
comments: false
youtube: e3rKYipvdJo
---

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

This is where we started before the episode

```R
library(tidyverse)
library(readxl)

metadata <- read_excel("raw_data/schubert.metadata.xlsx")

otu_counts <- read_tsv("raw_data/schubert.subsample.shared")

taxonomy <- read_tsv("raw_data/schubert.cons.taxonomy")
```


## Installations

* [See video](https://www.youtube.com/watch?v=D6CunpqF04E) for instructions on getting set up
* [R](https://r-project.org)
* [RStudio](https://rstudio.com)
* [Raw data](https://github.com/riffomonas/raw_data/releases/latest)
