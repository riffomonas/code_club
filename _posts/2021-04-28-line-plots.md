---
layout: post
title: "Using geom_line to change the appearance of a line plot with ggplot2 in R (CC097)"
blurb: "Creating attractive ROC curves"
author: "PD Schloss"
date: 2021-04-28 11:30
comments: false
youtube: XSRO4VKD-pc
---

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

This is where we started before the episode

```R
library(tidyverse)
library(readxl)
library(glue)
library(ggtext)

metadata <- read_excel(path="raw_data/schubert.metadata.xlsx", na="NA") %>%
  mutate(disease_stat = factor(disease_stat,
                               levels=c("NonDiarrhealControl",
                                        "DiarrhealControl",
                                        "Case")
                               )
  )

alpha_diversity <- read_tsv("raw_data/schubert.groups.ave-std.summary") %>%
  filter(method == "ave") %>%
  select(-label, -method)

disease_invsimpson <- inner_join(metadata, alpha_diversity,
                             by=c('sample_id'='group')) %>%
	select(disease_stat, invsimpson)


get_roc_data <- function(negative, positive){

  disease_invsimpson %>%
    filter(disease_stat == negative | disease_stat == positive) %>%
    mutate(disease_stat = recode(disease_stat,
                                 "{negative}" := FALSE,
                                 "{positive}" := TRUE)) %>%
    mutate(sens_spec = map(invsimpson, get_sens_spec, .)) %>%
    unnest(sens_spec) %>%
    mutate(comparison = glue("{negative}_{positive}"))

}

get_sens_spec <- function(x, data){

  predicted <- x > data$invsimpson

  tp <- sum(predicted & data$disease_stat)
  tn <- sum(!predicted & !data$disease_stat)
  fp <- sum(predicted & !data$disease_stat)
  fn <- sum(!predicted & data$disease_stat)

  specificity <- tn / (tn + fp)
  sensitivity <- tp / (tp + fn)


  tibble("sensitivity" = sensitivity, "specificity"=specificity)
}

roc_data <- bind_rows(
    get_roc_data("NonDiarrhealControl", "DiarrhealControl"),
    get_roc_data("NonDiarrhealControl", "Case"),
    get_roc_data("DiarrhealControl", "Case")
  ) %>%
  arrange(invsimpson)

roc_data %>%
  ggplot(aes(x=1-specificity, sensitivity, color=comparison)) +
  geom_abline(slope=1, intercept=0, color="gray") +
  geom_line() +
  theme_classic()

ggsave("schubert_roc_curves.tiff", width=7, height=4)
```

## Installations

* [See video](https://www.youtube.com/watch?v=D6CunpqF04E) for instructions on getting set up
* [R](https://r-project.org)
* [RStudio](https://rstudio.com)
* [Raw data](https://github.com/riffomonas/raw_data/releases/latest)
