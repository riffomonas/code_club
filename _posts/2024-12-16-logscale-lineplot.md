---
layout: post
title: "Adding log-scaled tick marks to an axis with ggplot2 in R (CC325)"
blurb: "Assembling figure using patchwork"
author: "PD Schloss"
date: 2024-12-16 8:00
comments: false
youtube: DXsF5jMHD40
start_hash: 
end_hash: 
---

Pat re-imagines a figure published in mBio as a line plot with data using the geom_jitter and stat_summary functions from ggplot2 in R. Along the way he uses filter, summarize, geom_errorbar, geom_line, labs, scale_color_manual, scale_y_log10, guide_axis_logticks, coord_cartesian, and theme. The original paper is in an open access journal and can be found at https://journals.asm.org/doi/10.1128/mbio.01206-24. The newsletter describing how he would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/thinking-through-how-to-plot-data-on-a-log-scaled-y-axis-with-ggplot2-in-r).

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)
library(ggtext)
library(glue)
library(patchwork)

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
                     rep(490, 6),
                     rnorm(12, mean = 5e6, sd = 2e6),
                     rnorm(3, mean = 5e6, sd = 2e6),
                     rnorm(3, mean = 1e3, sd = 5e2),
                     rep(490, 6),
                     
                     rnorm(3, mean = 5e6, sd = 1e6), #Ofloxacin
                     rnorm(3, mean = 2e3, sd = 8e2),
                     rep(490, 6),
                     rnorm(12, mean = 5e6, sd = 2e6),
                     rnorm(3, mean = 5e6, sd = 1e6),
                     rep(490, 9),
                     
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


plot_drug_data <- function(drug_name){

  fig2c_data %>%
    filter(drug == drug_name) %>%
    summarize(mean = mean(density),
              sd = sd(density),
              .by = c(strain, time)) %>%
    ggplot(aes(x = time, y = mean, color = strain,
               ymin = mean - sd, ymax = mean + sd)) +
    geom_errorbar(color = "black", width = 0.2, linewidth = 1) +
    geom_line(linewidth = 2) +
    labs(x = "Time (h)",
         y = "CFU/mL",
         color = NULL,
         title = drug_name) +
    scale_color_manual(breaks = c("PA01", "mutant", "mutant_arabinose"),
                       values = c("black", "pink", "darkred"),
                       labels = c("PA01",
                                  "&Delta;*iscU* P<sub>ara</sub>*iscU*",
                                  "&Delta;*iscU* P<sub>ara</sub>*iscU* (+0.5% ARA)")) +
    scale_y_log10(breaks = c(1e2, 1e3, 1e4, 1e5, 1e6),
                  labels = glue("10<sup>{2:6}</sup>"),
                  guide = guide_axis_logticks(
                    short = 1.5, mid = 1.5, long = 2.25)
                  ) +
    coord_cartesian(ylim = c(500, 1e7), expand = FALSE) +
    theme_classic() +
    theme(
      legend.text = element_markdown(size = 18, margin = margin(l = 10, r = 30)),
      legend.key.width = unit(40, "pt"),
      axis.line = element_line(linewidth = 1),
      axis.ticks = element_line(linewidth = 1),
      axis.ticks.length.x = unit(5, "pt"),
      axis.text.x = element_markdown(color = "black", size = 18,
                                     margin = margin(t = 10)),
      axis.title.x = element_text(size = 18,
                                  margin = margin(t = 10)),
      axis.text.y = element_markdown(color = "black", size = 18),
      axis.title.y = element_text(size = 18),
      panel.grid.major.y = element_line(color = "gray"),
      plot.title = element_text(size = 18, hjust = 1, margin = margin(r = 20))
    )
  
}

gentamicin <- plot_drug_data("Gentamicin") +
  theme(plot.margin = margin(r = 40)
)

ofloxacin <- plot_drug_data("Ofloxacin")

meropenem <- plot_drug_data("Meropenem") +
  theme(plot.margin = margin(r = 40, t = 20)
  )

colistin <- plot_drug_data("Colistin") +
  theme(plot.margin = margin(t = 20)
  )


(gentamicin | ofloxacin) /
  (meropenem | colistin) +
  plot_layout(guides = "collect") &
  theme(
    legend.position = "bottom",
    legend.margin = margin(t = 20, b = -8)
    )

ggsave("iron_sulfur_resistance.png", width = 9.87, height = 8)
```