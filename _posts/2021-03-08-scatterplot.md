---
layout: post
title: "Critiquing and improving a published ordination"
blurb: "Creating and improving a scatterplot"
author: "PD Schloss"
date: 2021-03-08 11:45
comments: false
youtube: xijDvx-J1jo
---

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

This is where we started before the episode

```R
library(tidyverse)
library(readxl)

nmds <- read_tsv(file="raw_data/schubert.braycurtis.nmds_2d.axes")
metadata <- read_excel(path="raw_data/schubert.metadata.xlsx")
metadata_nmds <- inner_join(metadata, nmds, by=c('sample_id'='group')) %>%
  mutate(disease_stat = factor(disease_stat,
                               levels=c("DiarrhealControl",
                                        "Case",
                                        "NonDiarrhealControl")
                               )
         )
```

## Installations

* [See video](https://www.youtube.com/watch?v=D6CunpqF04E) for instructions on getting set up
* [R](https://r-project.org)
* [RStudio](https://rstudio.com)
* [Raw data](https://github.com/riffomonas/raw_data/releases/latest)
