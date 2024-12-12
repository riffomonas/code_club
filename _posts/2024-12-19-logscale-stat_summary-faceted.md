---
layout: post
title: "Faceted log-scaled line plot with error bars using stat_summary from ggplot2 in R (CC326)"
blurb: "Assembling figure using facet_wrap"
author: "PD Schloss"
date: 2024-12-19 8:00
comments: false
youtube: 6kkkxJScpLE
start_hash: 
end_hash: 
---

Pat does a makeover of a figure published in mBio as a line plot with data using the stat_summary and facet_wrap functions from ggplot2 in R. Along the way he also uses geom_hline, coord_cartesian, scale_y_log10, guide_axis_logticks, scale_color_manual, labs, and theme. The original paper is in an open access journal and can be found at https://journals.asm.org/doi/10.1128/mbio.01206-24. The newsletter describing how he would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/thinking-through-how-to-plot-data-on-a-log-scaled-y-axis-with-ggplot2-in-r).

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)
library(glue)
library(ggtext)

set.seed(19760620)

fig2c_data <- tibble(
  drug = rep(c("Gentamicin", "Ofloxacin", "Meropenem", "Colistin"), each = 36),
  strain = rep(rep(c("PA01", "mutant", "mutant_arabinose"), each = 12),
               times = 4),
  rep = rep(1:3, times = 48),
  time = rep(rep(c(0, 1, 2, 3), each = 3), times = 12)
)  %>%
  mutate(density = c(rnorm(3, mean = 5e6, sd = 2e6), #Gentamicin
                     rnorm(3, mean = 1e3, sd = 3e2),
                     rep(100, 6),
                     rnorm(12, mean = 5e6, sd = 2e6),
                     rnorm(3, mean = 5e6, sd = 2e6),
                     rnorm(3, mean = 1e3, sd = 5e2),
                     rep(100, 6),
                     
                     rnorm(3, mean = 5e6, sd = 1e6), #Ofloxacin
                     rnorm(3, mean = 2e3, sd = 8e2),
                     rep(100, 6),
                     rnorm(12, mean = 5e6, sd = 2e6),
                     rnorm(3, mean = 5e6, sd = 1e6),
                     rep(100, 9),
                     
                     rnorm(3, mean = 5e6, sd = 2e6), #Meromenem
                     rnorm(3, mean = 5e5, sd = 4e5),
                     rnorm(3, mean = 1e5, sd = 4e4),
                     rnorm(3, mean = 6e4, sd = 2e4),
                     rnorm(12, mean = 5e6, sd = 2e6),
                     rnorm(3, mean = 5e6, sd = 1e6),
                     rnorm(3, mean = 5e5, sd = 1e5),
                     rnorm(3, mean = 1e5, sd = 2e4),
                     rnorm(3, mean = 6e4, sd = 1e4),
                     
                     rnorm(3, mean = 5e6, sd = 2e6), #Colistin
                     rnorm(3, mean = 5e3, sd = 3e3),
                     rnorm(3, mean = 1e3, sd = 4e2),
                     rnorm(3, mean = 8e2, sd = 3e2),
                     rnorm(3, mean = 6e6, sd = 2e6),
                     rnorm(3, mean = 6e3, sd = 3e3),
                     rnorm(3, mean = 2e3, sd = 3e2),
                     rnorm(3, mean = 1e3, sd = 3e2),
                     rnorm(3, mean = 5e6, sd = 2e6),
                     rnorm(3, mean = 5e3, sd = 3e3),
                     rnorm(3, mean = 1e3, sd = 4e2),
                     rnorm(3, mean = 8e2, sd = 3e2))
  )


fig2c_data %>%
  mutate(drug = factor(drug, levels = c("Gentamicin", "Ofloxacin",
                                        "Meropenem", "Colistin"))) %>%
  ggplot(aes(x = time, y = density, color = strain)) +
  geom_hline(yintercept = 500, linewidth = 0.2, linetype = "dashed") +
  stat_summary(geom = "line", 
               position = position_dodge(width = 0.1), 
               fun.data = mean_sdl, fun.args = list(mult = 1),
               linewidth = 0.75) +
  stat_summary(geom = "linerange",
               position = position_dodge(width = 0.1), 
               fun.data = mean_sdl, fun.args = list(mult = 1),
               show.legend = FALSE) +
  facet_wrap(~drug, nrow = 2, ncol = 2) +
  coord_cartesian(ylim = c(110, 1e7)) +
  scale_y_log10(
    breaks = c(1e2, 1e3, 1e4, 1e5, 1e6, 1e7),
    labels = glue("10<sup>{2:7}</sup>"),
    guide = guide_axis_logticks(long = 2.25, mid = 1, short = 1),
    expand = c(0, 0)
  ) +
  scale_color_manual(
    name = NULL,
    breaks = c("PA01", "mutant", "mutant_arabinose"),
    values = c("black", "pink", "darkred"), 
    labels = c("PA01",
               "&Delta;*iscU* P<sub>ara</sub>*iscU*",
               "&Delta;*iscU* P<sub>ara</sub>*iscU* (+0.5% ARA)")
  ) +
  labs(x = "Time (h)",
       y = "Cell density (CFU/mL)") +
  theme_classic() +
  theme(
    legend.position = "bottom",
    legend.text = element_markdown(margin = margin(l = 5, r = 10)),
    legend.margin = margin(0, 0, 0, 0),
    legend.box.margin = margin(0, 0, 0, 0),
    legend.key.height = unit(10, "pt"),
    axis.text = element_markdown(color = "black"),
    plot.margin = margin(3, 3, 3, 3),
    panel.grid.major.y = element_line(color = "gray", linewidth = 0.15),
    strip.background = element_blank(),
    strip.text = element_text(hjust = 0)
  )

ggsave("iron_sulfur_resistance.png", width = 5, height = 4)
```