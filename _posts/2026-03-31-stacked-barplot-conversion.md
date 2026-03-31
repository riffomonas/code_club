---
layout: post
title: "What would be more effective than a stacked bar plot? (CC412)"
blurb: "Anything...?"
author: "PD Schloss"
date: 2026-03-31 12:00
comments: false
youtube: fKpirOiva1c
start_hash: 
end_hash: 
---

In this livestream, Pat recreates and then refactors a staacked barplot from a paper published in the journal Nature Microbiology. He created the figure using R, readxl, dplyr, ggplot2, ggtext, glue, and other tools from the tidyverse. The functions he used from these packages included aes, annotate, coord_cartesian, download.file, element_blank, element_markdown, element_text, facet_wrap, factor, filter, geom_col, geom_hline, geom_point, ggplot, ggsave, guide_legend, if_else, labs, library, list, margin, mutate, paste, pivot_longer, position_jitter, read_excel, rep, rev, scale_alpha_manual, scale_color_manual, scale_fill_manual, scale_x_continuous, scale_x_discrete, scale_y_continuous, separate_wider_delim, seq, stat_summary, str_remove, str_replace, sum, summarize, theme, theme_classic, and unit. The original Nature Microbiology article can be [found here](https://www.nature.com/articles/s41564-026-02295-6). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```R
library(tidyverse)
library(readxl)
library(glue)
library(ggtext)

url <- "https://static-content.springer.com/esm/art%3A10.1038%2Fs41564-026-02295-6/MediaObjects/41564_2026_2295_MOESM8_ESM.xls"
download.file(url, "figure_4.xls")

week_rep <- c("2wk-2", "2wk-3", "4wk-1", "4wk-2", "6wk-1",
  "6wk-2", "8wk-1", "8wk-2",	"8wk-3", "10wk-1",
  "10wk-2", "12wk-1", "12wk-2")

week_label <- str_remove(week_rep, "-.*")

column_names <- paste(rep(week_rep, 2),
                      rep(c("wo_ldp", "w_ldp"), each = 13)
                      )

phyla_names <- rev(c("Bacteroidota", "Bacillota", "Pseudomonadota", 
                 "Actinomycetota", "Campylobacterota", 
                 "Spirochaetota", "Deferribacterota", 
                 "Chlamydiota", "Thermodesulfobacteriota",
                 "Other"))

pretty_phyla <- str_replace(phyla_names, "Thermodesulfobacteriota",
                            "Thermodesulfobacteriota")

data <- read_excel("figure_4.xls", sheet = "Figure 4c", range = "B5:AB15",
           col_names = c("phylum", column_names)) %>%
  pivot_longer(-phylum) %>%
  separate_wider_delim(name, delim = " ", names = c("week", "treatment")) %>%
  mutate(phylum = str_remove(phylum, "p__"),
         phylum = if_else(phylum %in% phyla_names, phylum, "Other")) %>%
  summarize(value = sum(value), .by = c(phylum, week, treatment)) %>%
  mutate(phylum = factor(phylum, levels = phyla_names,
                         labels = pretty_phyla),
         treatment = factor(treatment, levels = c("wo_ldp", "w_ldp"),
                            labels = c("Without Low\nDose Penicilin",
                                       "With Low\nDose Penicilin")),
         week = factor(week, levels = week_rep))

data %>%
  ggplot(aes(x = week, y = value * 100, fill = phylum, alpha = phylum,
             color = phylum)) +
  geom_col() + 
  facet_wrap(~treatment) +
  labs(x = NULL, y = "Relative abundance\nat phylum level (%)") +
  scale_fill_manual(
    name = "Gram<sup>+</sup> bacteria",
    breaks = c("Bacillota", "Actinomycetota"),
    values = c("Bacillota" = "#4CBBD5", "Actinomycetota" = "#3C5488",
               "Pseudomonadota" = "#00A087", "Spirochaetota" = "#DC0000",
               "Deferribacterota" = "#91D1C2", "Bacteroidota" = "#E64A35",
               "Campylobacterota" = "#F39B7E", "Chlamydiota" = "#B09C85",
               "Thermodesulfobacteriota" = "#8391B4", "Other" = "darkgray"),
    guide = guide_legend(order = 1)
  ) +
  scale_alpha_manual(
    name = "Gram<sup>-</sup> bacteria",
    values = rep(1, 10),
    breaks = c("Pseudomonadota", "Spirochaetota", "Deferribacterota",
               "Bacteroidota", "Campylobacterota", "Chlamydiota",
               "Thermodesulfobacteriota"),
    guide = guide_legend(
      order = 2,
      override.aes = list(
        fill = c("Pseudomonadota" = "#00A087", "Spirochaetota" = "#DC0000",
        "Deferribacterota" = "#91D1C2", "Bacteroidota" = "#E64A35",
        "Campylobacterota" = "#F39B7E", "Chlamydiota" = "#B09C85",
        "Thermodesulfobacteriota" = "#8391B4")
    ))
  ) +
  scale_color_manual(
    name = NULL,
    values = c("Bacillota" = NA, "Actinomycetota" = NA, "Pseudomonadota" = NA,
               "Spirochaetota" = NA, "Deferribacterota" = NA,
               "Bacteroidota" = NA, "Campylobacterota" = NA,
               "Chlamydiota" = NA, "Thermodesulfobacteriota" = NA,
               "Other" = NA),
    breaks = "Other",
    guide = guide_legend(
      order = 3,
      override.aes = list(
        fill = c("Other" = "darkgray")
      ))
  ) +
  scale_x_discrete(
    breaks = week_rep,
    labels = week_label
  ) +
  scale_y_continuous(expand = c(0, 0)) +
  coord_cartesian(ylim = c(0, 100), xlim = c(1, 13), clip = "off") +
  annotate(geom = "text", x = -1.1, y = -8, label = "Age", layout = 1,
           size = 6, size.unit = "pt") +
  theme(
    text = element_text(color = "black"),
    plot.margin = margin(2, 2, 2, 2),
    panel.grid = element_blank(),
    panel.background = element_blank(),
    
    legend.key.height = unit(8, "pt"),
    legend.key.width = unit(8, "pt"),
    legend.text = element_text(size = 7, lineheight = 0.7, 
                               margin = margin(l = 2)),
    legend.title = element_markdown(size = 7, margin = margin(b = 2)),
    legend.box.spacing = unit(0, "pt"),
    legend.margin = margin(0, 0, 0, 3),
    legend.spacing.y = unit(5, "pt"),
    
    axis.ticks = element_blank(),
    axis.text.x = element_text(angle = 90, hjust = 1, vjust = 0.5, size = 6),
    axis.text.y = element_text(size = 6),
    axis.title.y = element_text(size = 7),
    
    strip.text = element_text(size = 7, margin = margin(b = 2)),
    strip.background = element_blank(),
  )

ggsave("figure_4c.png", width = 3.75, height = 2.0)


data %>%
  mutate(week_dbl = str_remove(week, "wk-\\d+") %>% as.numeric) %>%
  filter(phylum %in% c("Bacteroidota", "Bacillota", "Pseudomonadota", 
                       "Actinomycetota")) %>% 
  ggplot(aes(x = week_dbl, y = 100 * value, color = treatment)) +
  geom_hline(yintercept = 0) +
  geom_point(position = position_jitter(width = 0.4, seed = 19760620),
             show.legend = TRUE) +
  stat_summary(geom = "line", fun = mean, 
               linewidth = 0.5, 
               show.legend = FALSE) +
  facet_wrap(~phylum, axes = "all", axis.labels = "margins") +
  coord_cartesian(ylim = c(0, 100), 
                  xlim = c(1, 13),
                  expand = FALSE, clip = "off") +
  scale_x_continuous(breaks = seq(2, 12, 2)) +
  scale_color_manual(
    name = "Penicilin",
    values = c("Without Low\nDose Penicilin" = "gray",
               "With Low\nDose Penicilin" = "dodgerblue"),
    labels = c("No", "Yes")
  ) +
  labs(x = "Age (weeks)", y = "Relative abundance\nat phylum level (%)") +
  theme_classic(base_size = 7, base_line_size = 0.2) +
  theme(
    panel.background = element_blank(),
    strip.background = element_blank(),
    axis.line.x = element_blank(), 
    legend.position = "inside",
    legend.position.inside = c(0.95, 0.85),
    legend.key.size = unit(6, "pt"),
    legend.title = element_text(size = 6, hjust = 0.5, margin = margin(b = 2))
  )

ggsave("figure_4c_redo.png", width = 3.75, height = 2.0)
```