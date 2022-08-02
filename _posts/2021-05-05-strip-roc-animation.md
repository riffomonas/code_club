---
layout: post
title: "Animate two synchronized figures in R with gganimate, magick, and ggplot2 (CC100)"
blurb: "Aminating graphs in R"
author: "PD Schloss"
date: 2021-05-05 11:30
comments: false
youtube: sZYt6_AqKUc
---

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

This is where we started before the episode

```R
library(tidyverse)
library(readxl)
library(glue)
library(ggtext)
library(patchwork)

set.seed(19760620)

metadata <- read_excel(path="raw_data/schubert.metadata.xlsx", na="NA")

alpha_diversity <- read_tsv("raw_data/schubert.groups.ave-std.summary") %>%
  filter(method == "ave") %>%
  select(-label, -method)

healthy_color <- "#BEBEBE"
diarrhea_color <- "#0000FF"
case_color <- "#FF0000"

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


pretty_names <- c("NonDiarrhealControl_Case" = "<strong style='color:#BEBEBE;'>Healthy</strong> vs. <br><strong style='color:#FF0000;'>*C. difficile* positive</strong>",
                  "NonDiarrhealControl_DiarrhealControl" = "<strong style='color:#BEBEBE;'>Healthy</strong> vs.<br><strong style='color:#0000FF;'>*C. difficile* negative</strong>",
                  "DiarrhealControl_Case" = "*C. difficile*<br><strong style='color:#0000FF;'>negative</strong> vs. <br><strong style='color:#FF0000;'>positive</strong>")


test <- "NonDiarrhealControl_Case"

test_data <- roc_data %>%
  filter(comparison == test) %>%
  separate(comparison, into=c("negative", "positive"), remove=FALSE) %>%
  mutate(disease_stat = if_else(disease_stat, positive, negative),
					disease_stat = factor(disease_stat,
                               levels=c("NonDiarrhealControl",
                                        "DiarrhealControl",
                                        "Case")
                               )
  )


diversity_plot <- test_data %>%
  ggplot(aes(x=disease_stat, y=invsimpson, fill=disease_stat)) +
  geom_jitter(show.legend=FALSE, width=0.25, shape=21, color="black") +
  labs(x=NULL,
       y="Inverse Simpson Index") +
  scale_x_discrete(breaks=c("NonDiarrhealControl","DiarrhealControl","Case"),
                   labels=c(glue("Healthy"),
                            glue("Diarrhea and<br>*C.difficile* negative"),
                            glue("Diarrhea and<br>*C.difficile* positive"))
  ) +
  scale_fill_manual(name=NULL,
                    breaks=c("NonDiarrhealControl","DiarrhealControl","Case"),
                    labels=c("Healthy",
                             "Diarrhea and<br>*C.difficile* negative",
                             "Diarrhea and<br>*C.difficile* positive"),
                    values=c(healthy_color, diarrhea_color, case_color)) +
  scale_y_continuous(limits=c(0,34)) +
  theme_classic() +
  theme(axis.text.x = element_markdown())


roc_plot <- test_data %>%
  ggplot(aes(x=1-specificity, sensitivity, group=comparison)) +
  geom_abline(slope=1, intercept=0, color="gray") +
  geom_line(size = 1, linejoin="round", color="black") +
  labs(x="1-Specificity",
       y="Sensitivity") +
  coord_fixed() +
  theme_classic() +
  theme(
    legend.text = element_markdown(),
    legend.key.height = unit(25, "pt"),
    legend.position = c(0.8, 0.2)
  ) +
  geom_richtext(data=tibble(x=0.75, y=0.15, label=pretty_names[test]),
                aes(x=x, y=y, label=label), inherit.aes = FALSE,
                fill=NA, label.color=NA, size=5)

(diversity_plot | roc_plot) + plot_layout(widths=c(1, 2))

ggsave("side_by_side.tiff", width=6.5, height=4)
```


## Installations

* [See video](https://www.youtube.com/watch?v=D6CunpqF04E) for instructions on getting set up
* [R](https://r-project.org)
* [RStudio](https://rstudio.com)
* [Raw data](https://github.com/riffomonas/raw_data/releases/latest)
