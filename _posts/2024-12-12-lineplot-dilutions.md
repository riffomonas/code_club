---
layout: post
title: "Plotting a dilution series using ggplot2 in R with geom_jitter and stat_summary (CC324)"
blurb: "Fun with stat_summary"
author: "PD Schloss"
date: 2024-12-12 8:00
comments: false
youtube: S7HpVmYqmMI
start_hash: 
end_hash: 
---

Pat re-imagines a figure published in mSystems as a line plot with data using the geom_jitter and stat_summary functions from ggplot2 in R. Along the way he uses stat_summary, geom_jitter, geom_vline, scale_x_log10, scale_color_manual, labs, and theme. The original paper is in an open access journal and can be found at https://journals.asm.org/doi/10.1128/msystems.00992-24. The newsletter describing how he would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/reverse-engineering-a-dilution-series-of-box-plots).

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
  copper = rep(rep(c(0, 0.003, 0.01, 0.03, 0.3, 30, 300), each = 3), 2),
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
)

cu_fluor %>%
  mutate(copper = if_else(copper == 0, 3e-4, copper)) %>%
  ggplot(aes(x = copper, y = fluorescence, color = treatment)) +
  stat_summary(geom = "line", fun = median,
               position = position_dodge(width = 0.25),
               linewidth = 1) +
  geom_jitter(alpha = 0.25, #size = 0.5,
              position = position_jitterdodge(jitter.width = 0.05,
                                              dodge.width = 0.25)) +
  geom_vline(xintercept = 1e-3, color = "gray", linetype = "dashed",
             linewidth = 0.5) +
  scale_x_log10(limits = c(1e-4, 1e3), expand = c(0, 0),
                breaks = c(3e-4, 3e-3, 3e-2, 3e-1, 3, 30, 300),
                labels = c(0, 3e-3, 3e-2, 3e-1, 3, 30, 300)) +
  scale_color_manual(
    values = c("gray", "blue"),
    guide = guide_legend(override.aes = list(linewidth = 0.75))) +
  labs(color = NULL,
       x = "Copper concentration (\u03BCM)",
       y = "Fluorescence (% of 0 \u03BCM Cu)") +
  theme_classic() + 
  theme(legend.position = "inside",
        legend.position.inside = c(0.8, 0.6),
        legend.background = element_rect(color = "black"),
        legend.margin = margin(1, 2, 1, 1),
        legend.key.height = unit(10, "pt"))

ggsave("lineplot_dilutions.png", width = 5, height = 3)
```