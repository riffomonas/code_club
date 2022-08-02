---
layout: post
title: "Rotating axis labels and flipping coordiantes in R"
blurb: "Using coord_flip and guide_axis to improve the readability of a figure"
author: "PD Schloss"
date: 2021-04-21 11:30
comments: false
youtube: VNYpVwUMQ2o
---

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

This is where we started before the episode

```R
library(tidyverse)
library(readxl)
library(glue)
library(ggtext)

set.seed(19760620)

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


kt <- kruskal.test(invsimpson ~ disease_stat, data=metadata_alpha)

if(kt$p.value < 0.05){
  pt <- pairwise.wilcox.test(metadata_alpha$invsimpson,
                       g=metadata_alpha$disease_stat,
                       p.adjust.method = "BH")
}

strip_chart <- metadata_alpha %>%
  ggplot(aes(x=disease_stat, y=invsimpson, fill=disease_stat)) +
  stat_summary(fun = median, show.legend=FALSE, geom="crossbar") +
  geom_jitter(show.legend=FALSE, width=0.25, shape=21, color="black") +
  labs(x=NULL,
       y="Inverse Simpson Index") +
  scale_x_discrete(breaks=c("NonDiarrhealControl","DiarrhealControl","Case"),
                   labels=c(glue("Healthy (N={healthy_n})"),
                            glue("Diarrhea and *C.difficile* negative \\
                                 (N={diarrhea_n})"),
                            glue("Diarrhea and *C.difficile* positive \\
                                 (N={case_n})"))
                   ) +
  scale_fill_manual(name=NULL,
                     breaks=c("NonDiarrhealControl","DiarrhealControl","Case"),
                     labels=c("Healthy",
                              "Diarrhea and *C.difficile* negative",
                              "Diarrhea and *C.difficile* positive"),
                     values=c(healthy_color, diarrhea_color, case_color)) +
  theme_classic() +
  theme(axis.text.x = element_markdown())

strip_chart +
  geom_line(data=tibble(x=c(2, 3), y=c(23, 23)),
            aes(x=x, y=y),
            inherit.aes=FALSE) +
  geom_line(data=tibble(x=c(1, 2.5), y=c(33, 33)),
            aes(x=x, y=y),
            inherit.aes=FALSE) +
  geom_text(data=tibble(x=2.5, y=24),
            aes(x=x, y=y, label="n.s."),
            inherit.aes=FALSE) +
  geom_text(data=tibble(x=1.75, y=33.5),
            aes(x=x, y=y, label="*"), size=8,
            inherit.aes=FALSE)


ggsave("schubert_diversity.tiff", width=4.5, height=3.5)
```

## Installations

* [See video](https://www.youtube.com/watch?v=D6CunpqF04E) for instructions on getting set up
* [R](https://r-project.org)
* [RStudio](https://rstudio.com)
* [Raw data](https://github.com/riffomonas/raw_data/releases/latest)
