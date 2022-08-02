---
layout: post
title: "String manipulation in R with regular expressions using stringr and glue (CC111)"
blurb: "Removing underscores and applying italicization to axis labels"
author: "PD Schloss"
date: 2021-06-01 11:00
comments: false
youtube: u0yQfr7d0vs
---

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

This is where we started before the episode

```R
library(tidyverse)
library(readxl)
library(ggtext)

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
           sep=";")

otu_rel_abund <- inner_join(metadata, otu_counts, by="sample_id") %>%
  inner_join(., taxonomy, by="otu") %>%
  group_by(sample_id) %>%
  mutate(rel_abund = count / sum(count)) %>%
  ungroup() %>%
  select(-count) %>%
  pivot_longer(
    c("kingdom", "phylum", "class", "order", "family", "genus", "otu"),
    names_to="level",
    values_to="taxon") %>%
  mutate(disease_stat = factor(disease_stat,
                               levels=c("Case",
                                        "DiarrhealControl",
                                        "NonDiarrhealControl")))


taxon_rel_abund <- otu_rel_abund %>%
  filter(level=="genus") %>%
  group_by(disease_stat, sample_id, taxon) %>%
  summarize(rel_abund = 100*sum(rel_abund), .groups="drop")

taxon_pool <- taxon_rel_abund %>%
  group_by(disease_stat, taxon) %>%
  summarize(median=median(rel_abund), .groups="drop") %>%
  group_by(taxon) %>%
  summarize(pool = max(median) < 1,
            median = median(median),
            .groups="drop")

inner_join(taxon_rel_abund, taxon_pool, by="taxon") %>%
  mutate(taxon = if_else(pool, "Other", taxon)) %>%
  group_by(sample_id, disease_stat, taxon) %>%
  summarize(rel_abund = sum(rel_abund),
            median = min(median),
            .groups="drop") %>%
  mutate(taxon = factor(taxon),
         taxon = fct_reorder(taxon, median, .desc=FALSE)) %>%
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

ggsave("schubert_genus.tiff", width=7, height=6)
```


## Installations

* [See video](https://www.youtube.com/watch?v=D6CunpqF04E) for instructions on getting set up
* [R](https://r-project.org)
* [RStudio](https://rstudio.com)
* [Raw data](https://github.com/riffomonas/raw_data/releases/latest)
