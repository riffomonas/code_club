---
layout: post
title: "How to create a grouped plot of jittered data with the ggplot2 R package (CC108)"
blurb: "Using jitterplots to represent relative abundance data"
author: "PD Schloss"
date: 2021-05-24 11:30
comments: false
youtube: XOTP71UsBzA
---

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

This is where we started before the episode

```R
library(tidyverse)
library(readxl)
library(ggtext)
library(RColorBrewer)

metadata <- read_excel("raw_data/schubert.metadata.xlsx", na="NA") %>%
  select(sample_id, disease_stat) %>%
  drop_na(disease_stat)

otu_counts <- read_tsv("raw_data/schubert.subsample.shared") %>%
  select(Group, starts_with("Otu")) %>%
  rename(sample_id = Group) %>%
  pivot_longer(-sample_id, names_to="otu", values_to = "count")

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
  pivot_longer(c("kingdom", "phylum", "class", "order", "family", "genus", "otu"),
               names_to="level",
               values_to="taxon") %>%
  mutate(disease_stat = factor(disease_stat,
                               levels=c("NonDiarrhealControl",
                                        "DiarrhealControl",
                                        "Case")))


taxon_rel_abund <- otu_rel_abund %>%
  filter(level=="phylum") %>%
  group_by(disease_stat, sample_id, taxon) %>%
  summarize(rel_abund = sum(rel_abund), .groups="drop") %>%
  group_by(disease_stat, taxon) %>%
  summarize(mean_rel_abund = 100*mean(rel_abund), .groups="drop") %>%
  mutate(taxon = str_replace(taxon,
                             "(.*)_unclassified", "Unclassified<br>*\\1*"),
         taxon = str_replace(taxon,
                             "^([^<]*)$", "*\\1*"))

taxon_pool <- taxon_rel_abund %>%
  group_by(taxon) %>%
  summarize(pool = max(mean_rel_abund) < 3,
            mean = mean(mean_rel_abund),
            .groups="drop")

inner_join(taxon_rel_abund, taxon_pool, by="taxon") %>%
  mutate(taxon = if_else(pool, "Other", taxon)) %>%
  group_by(disease_stat, taxon) %>%
  summarize(mean_rel_abund = sum(mean_rel_abund),
            mean = min(mean),
            .groups="drop") %>%
  mutate(taxon = factor(taxon),
         taxon = fct_reorder(taxon, mean, .desc=TRUE)) %>%
  ggplot(aes(x=taxon, y=mean_rel_abund, fill=disease_stat)) +
  geom_col(position = position_dodge()) +
  scale_fill_manual(name=NULL,
                   breaks=c("NonDiarrhealControl",
                            "DiarrhealControl",
                            "Case"),
                   labels=c("Healthy",
                            "Diarrhea,<br>*C. difficile* negative",
                            "Diarrhea,<br>*C. difficile* positive"),
                   values=c("gray", "blue", "red")) +
  scale_y_continuous(expand=c(0, 0)) +
  labs(x=NULL,
       y="Mean Relative Abundance (%)") +
  theme_classic() +
  theme(axis.text.x = element_markdown(),
        legend.text = element_markdown(),
        legend.position = c(0.8, 0.6))

ggsave("schubert_phylum.tiff", width=6, height=4)
```


## Installations

* [See video](https://www.youtube.com/watch?v=D6CunpqF04E) for instructions on getting set up
* [R](https://r-project.org)
* [RStudio](https://rstudio.com)
* [Raw data](https://github.com/riffomonas/raw_data/releases/latest)
