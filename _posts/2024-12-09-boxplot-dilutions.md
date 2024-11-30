---
layout: post
title: "Recreating published boxplots from a dilution series using ggplot2 in R (CC323)"
blurb: "Box plots galore"
author: "PD Schloss"
date: 2024-12-09 8:00
comments: false
youtube: nEA1guRuO7g
start_hash: 
end_hash: 
---

Pat recreates a figure published in mSystems that showed boxplots for two treatment groups along a dilution series using tools from the ggplot2 R package. Along the way he uses geom_boxplot, geom_jitter, annotate, labs, coord_cartesian, scale_color_manual, and theme. The original paper is in an open access journal and can be found at https://journals.asm.org/doi/10.1128/msystems.00992-24. The newsletter describing how he would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/reverse-engineering-a-dilution-series-of-box-plots).

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
# Figure 5D from
#
# Bayne C, Boutard M, Zaplana T, Tolonen AC. 0. L-tryptophan and copper
# interactions linked to reduced colibactin genotoxicity in pks+ Escherichia
# coli. mSystems 0:e00992-24. https://doi.org/10.1128/msystems.00992-24

library(tidyverse)
set.seed(19760620)

cu_fluor <- tibble(
  treatment = c(rep("7H4M", 21), rep("ClbP-17", 21)),
  copper = as.character(
    rep(rep(c(0, 0.003, 0.01, 0.03, 0.3, 30, 300), each = 3), 2)),
  fluorescence = c(101, 100, 99,
                   100, 98, 97,
                   101, 100, 95,
                   98, 97, 99,
                   100, 98, 97,
                   98, 97, 96,
                   98, 97, 95,
                   102, 101, 98,
                   92, 88, 85,
                   88, 85, 85,
                   85, 85, 84,
                   71, 65, 64,
                   63, 60, 58,
                   63, 62, 61)
) %>%
  mutate(copper_treatment = paste(copper, treatment))

cu_fluor %>%
  ggplot(aes(x = copper_treatment, y = fluorescence, color = treatment)) +
  geom_boxplot(linewidth = 0.75) +
  geom_jitter(color = "black") +
  annotate("text",
           x = c(-0.4, seq(1.5, 13.5, 2)),
           y = 52,
           label = c("Cu (\u03BCM)", 0, 0.003, 0.01, 0.03, 0.3, 30, 300),
           size = 12, size.unit = "pt")+
  annotate("segment",
           x = seq(2.5, 14.5, 2),
           xend = seq(2.5, 14.5, 2),
           y = 55,
           yend = 53.5,
           linewidth = 1.25) +
  theme_bw() +
  labs(color = NULL,
       x = NULL,
       y = "Fluorescence (% no Cu)" ) +
  coord_cartesian(xlim = c(0.5, 14.7), ylim = c(55, 105), clip = "off",
                  expand = FALSE) +
  scale_color_manual(values = c("#ED570D", "#68A2AC")) +
  theme(
    panel.grid = element_blank(),
    legend.position = "inside",
    legend.position.inside = c(0.85, 0.6),
    legend.background = element_rect(color = "black"),
    legend.margin = margin(1, 2, 1, 2),
    legend.text = element_text(size = 12),
    legend.key.size = unit(25, units = "pt"),
    axis.title = element_text(size = 16),
    axis.text.y = element_text(size = 12, margin = margin(r = 5),
                               color = "black"),
    axis.ticks.y = element_line(linewidth = 1.25),
    axis.ticks.length.y = unit(7, "pt"),
    axis.ticks.x = element_blank(),
    axis.text.x = element_blank(),
    plot.margin = margin(5, 5, 20, 5)
  )

ggsave("boxplot_dilutions.png", width = 7.5, height = 3.68)
```