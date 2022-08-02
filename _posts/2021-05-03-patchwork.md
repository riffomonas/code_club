---
layout: post
title: "Learning to use the patchwork R package (how to learn a package in general) (CC099)"
blurb: "Creating multipanel figures in R"
author: "PD Schloss"
date: 2021-05-03 11:30
comments: false
youtube: 2o1YDUKyhu0
---

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

This is where we started before the episode

```R
library(tidyverse)
library(readxl)
library(glue)
library(ggtext)
library(cowplot)

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
                   labels=c(glue("Healthy<br>(N={healthy_n})"),
                            glue("Diarrhea and<br>*C.difficile* negative<br>\\
                                 (N={diarrhea_n})"),
                            glue("Diarrhea and<br>*C.difficile* positive<br>\\
                                 (N={case_n})"))
  ) +
  scale_fill_manual(name=NULL,
                    breaks=c("NonDiarrhealControl","DiarrhealControl","Case"),
                    labels=c("Healthy",
                             "Diarrhea and<br>*C.difficile* negative",
                             "Diarrhea and<br>*C.difficile* positive"),
                    values=c(healthy_color, diarrhea_color, case_color)) +
  theme_classic() +
  theme(axis.text.x = element_markdown(size=6)) +
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

get_roc_curve <- function(test){

  roc_data %>%
    filter(comparison == test) %>%
    ggplot(aes(x=1-specificity, sensitivity, group=comparison)) +
    geom_abline(slope=1, intercept=0, color="gray") +
    geom_line(size = 1, linejoin="round", color="black") +
    labs(x="1-Specificity",
         y="Sensitivity") +
    theme_classic() +
    theme(
      legend.text = element_markdown(),
      legend.key.height = unit(25, "pt"),
      legend.position = c(0.8, 0.2)
    ) +
    geom_richtext(data=tibble(x=0.75, y=0.15, label=pretty_names[test]),
                  aes(x=x, y=y, label=label), inherit.aes = FALSE,
                  fill=NA, label.color=NA)
  }

ndc_c <- get_roc_curve("NonDiarrhealControl_Case")
dc_c <- get_roc_curve("DiarrhealControl_Case")
ndc_dc <- get_roc_curve("NonDiarrhealControl_DiarrhealControl")

plot_grid(strip_chart, ndc_dc, ndc_c, dc_c,
          nrow=2, ncol=2, rel_widths = c(1, 1),
          labels=c("A", "B", "C", "D"))


ggsave("schubert_fig_1.tiff", width=6.5, height=6)
```


## Installations

* [See video](https://www.youtube.com/watch?v=D6CunpqF04E) for instructions on getting set up
* [R](https://r-project.org)
* [RStudio](https://rstudio.com)
* [Raw data](https://github.com/riffomonas/raw_data/releases/latest)
