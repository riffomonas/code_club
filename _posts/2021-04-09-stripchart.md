---
layout: post
title: "Creating a strip chart/jitter plot with a line to indicate the median"
blurb: "Combining geom_jitter and stat_summary"
author: "PD Schloss"
date: 2021-04-09 11:30
comments: false
youtube: tAVNv-Gz6NU
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

metadata_alpha <- inner_join(metadata, alpha_diversity,
                             by=c('sample_id'='group')
                             )

healthy_color <- "#BEBEBE"
diarrhea_color <- "#0000FF"
case_color <- "#FF0000"

disease_count <- metadata_alpha %>%
  count(disease_stat)

healthy_n <- disease_count %>%
  filter(disease_stat == "NonDiarrhealControl") %>%
  pull(n)

diarrhea_n <- disease_count %>%
  filter(disease_stat == "DiarrhealControl") %>%
  pull(n)

case_n <- disease_count %>%
  filter(disease_stat == "Case") %>%
  pull(n)

metadata_alpha %>%
  ggplot(aes(x=disease_stat, y=invsimpson, color=disease_stat)) +
  stat_summary(fun.data = median_hilow, show.legend=FALSE, geom="linerange",
               color="black") +
  stat_summary(fun.data = median_hilow, show.legend=FALSE, geom="point",
               size=3) +
  labs(x=NULL,
       y="Inverse Simpson Index") +
  scale_x_discrete(breaks=c("NonDiarrhealControl","DiarrhealControl","Case"),
                   labels=c(glue("Healthy<br>(N={healthy_n})"),
                            glue("Diarrhea and<br>*C.difficile* negative<br>\\
                                 (N={diarrhea_n})"),
                            glue("Diarrhea and<br>*C.difficile* positive<br>\\
                                 (N={case_n})"))
                   ) +
  scale_color_manual(name=NULL,
                     breaks=c("NonDiarrhealControl","DiarrhealControl","Case"),
                     labels=c("Healthy",
                              "Diarrhea and<br>*C.difficile* negative",
                              "Diarrhea and<br>*C.difficile* positive"),
                     values=c(healthy_color, diarrhea_color, case_color)) +
  theme_classic() +
  theme(axis.text.x = element_markdown())

ggsave("schubert_diversity.tiff", width=4.5, height=3.5)
```

## Installations

* [See video](https://www.youtube.com/watch?v=D6CunpqF04E) for instructions on getting set up
* [R](https://r-project.org)
* [RStudio](https://rstudio.com)
* [Raw data](https://github.com/riffomonas/raw_data/releases/latest)
