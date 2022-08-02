---
layout: post
title: "How to dynamically add significance bars and stars to a figure in ggplot2 (CC113)"
blurb: "Indicating significance"
author: "PD Schloss"
date: 2021-06-08 11:00
comments: false
youtube: -DfCitMqdUc
---

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

This is where we started before the episode

```R
library(tidyverse)
library(readxl)
library(ggtext)
library(glue)
library(broom)

metadata <- read_excel("raw_data/schubert.metadata.xlsx", na="NA") %>%
  select(sample_id, disease_stat) %>%
  drop_na(disease_stat)

otu_counts <- read_tsv("raw_data/schubert.subsample.shared") %>%
  select(Group, starts_with("Otu")) %>%
  rename(sample_id = Group) %>%
  pivot_longer(-sample_id, names_to="otu", values_to = "count")

nseqs_per_sample <- otu_counts %>%
  group_by(sample_id) %>%
  summarize(N = sum(count), .groups="drop") %>%
  count(N) %>%
  pull(N)

stopifnot(length(nseqs_per_sample) == 1)

lod <- 100* 1/nseqs_per_sample

taxonomy <- read_tsv("raw_data/schubert.cons.taxonomy") %>%
  select("OTU", "Taxonomy") %>%
  rename_all(tolower) %>%
  mutate(taxonomy = str_replace_all(taxonomy, "\\(\\d+\\)", ""),
         taxonomy = str_replace(taxonomy, ";$", "")) %>%
  separate(taxonomy,
           into=c("kingdom", "phylum", "class", "order", "family", "genus"),
           sep=";") %>%
  mutate(pretty_otu = str_replace(string=otu,
                                  pattern="tu0*",
                                  replacement = "TU "),
         genus = str_replace(string=genus,
                             pattern="(.*)",
                             replacement="*\\1*"),
         genus = str_replace(string=genus,
                             pattern="\\*(.*)_unclassified\\*",
                             replacement="Unclassified<br>*\\1*"),
         taxon = glue("{genus}<br>({pretty_otu})")) %>%
  select(otu, taxon)

otu_rel_abund <- inner_join(metadata, otu_counts, by="sample_id") %>%
  inner_join(., taxonomy, by="otu") %>%
  group_by(sample_id) %>%
  mutate(rel_abund = 100*count / sum(count)) %>%
  ungroup() %>%
  select(-count) %>%
  mutate(disease_stat = factor(disease_stat,
                               levels=c("Case",
                                        "DiarrhealControl",
                                        "NonDiarrhealControl")))


taxon_pool <- otu_rel_abund %>%
  group_by(disease_stat, taxon) %>%
  summarize(median=median(rel_abund), .groups="drop") %>%
  group_by(taxon) %>%
  summarize(pool = max(median) < 0.5,
            median = max(median),
            .groups="drop")

otu_disease_rel_abund <- inner_join(otu_rel_abund, taxon_pool, by="taxon") %>%
  mutate(taxon = if_else(pool, "Other", as.character(taxon))) %>%
  group_by(sample_id, disease_stat, taxon) %>%
  summarize(rel_abund = sum(rel_abund),
            median = max(median),
            .groups="drop") %>%
  mutate(taxon = factor(taxon),
         taxon = fct_reorder(taxon, median, .desc=FALSE))


experiment_significance <- otu_disease_rel_abund %>%
  group_by(taxon) %>%
  nest() %>%
  mutate(experiment_tests = map(.x=data,
                                ~kruskal.test(rel_abund~disease_stat, data=.x) %>%
                                  tidy())) %>%
  unnest(experiment_tests) %>%
  mutate(p.experiment = p.adjust(p.value, method="BH")) %>%
  select(taxon, data, p.experiment) %>%
  filter(p.experiment < 0.05)

experiment_significance %>%
  mutate(pairwise_tests = map(.x=data,
                              ~pairwise.wilcox.test(x=.x$rel_abund,
                                                    g=.x$disease_stat,
                                                    p.adjust.method = "BH") %>%
                                tidy())) %>%
  unnest(pairwise_tests) %>%
  filter(p.value < 0.05) %>%
  select(taxon, group1, group2, p.value) %>%
  ungroup()

  mutate(rel_abund = if_else(rel_abund == 0,
                             2/3 * lod,
                             rel_abund)) %>%
  ggplot(aes(y=taxon, x=rel_abund, color=disease_stat)) +
  geom_vline(xintercept = lod, size=0.2) +
  stat_summary(fun.data=median_hilow, geom = "pointrange",
               fun.args=list(conf.int=0.5),
               position = position_dodge(width=0.6)) +
  coord_trans(x="log10") +
  scale_x_continuous(limits=c(NA, 100),
                     breaks=c(0.1, 1, 10, 100),
                     labels=c(0.1, 1, 10, 100)) +
  scale_color_manual(name=NULL,
                   breaks=c("NonDiarrhealControl",
                            "DiarrhealControl",
                            "Case"),
                   labels=c("Healthy",
                            "Diarrhea,<br>*C. difficile* negative",
                            "Diarrhea,<br>*C. difficile* positive"),
                   values=c("gray", "blue", "red")) +
  labs(y=NULL,
       x="Relative Abundance (%)") +
  theme_classic() +
  theme(axis.text.y = element_markdown(),
        legend.text = element_markdown(),
        # legend.position = c(0.8, 0.6),
        legend.background = element_rect(color="black", fill = NA),
        legend.margin = margin(t=-5, r=3, b=3)
        )

ggsave("schubert_otu.tiff", width=7, height=6)
```


## Installations

* [See video](https://www.youtube.com/watch?v=D6CunpqF04E) for instructions on getting set up
* [R](https://r-project.org)
* [RStudio](https://rstudio.com)
* [Raw data](https://github.com/riffomonas/raw_data/releases/latest)
