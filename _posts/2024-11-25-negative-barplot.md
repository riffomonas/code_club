---
layout: post
title: "Creating bar plot with error bars using ggplot2's stat_summary in R (CC318)"
blurb: "Descending barplot"
author: "PD Schloss"
date: 2024-11-25 8:00
comments: false
youtube: Oe5O4LRj2rc
start_hash: 
end_hash: 
---

Pat shows how to use stat_summary() and other functions from ggplot2 to create a descending barplot with error bars in R. He customizes the appearance using scale_x_discrete, scale_y_continuous, scale_fill_manual, geom_hline, and theme. The newsletter describing how I would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/some-thoughts-on-how-to-recreate-a-descending-bar-plot-with-ggplot2).

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```
library(tidyverse)
library(ggtext)
set.seed(19760620)

mean_sd <- function(x){

  tibble(y = mean(x),
         ymin = y - sd(x),
         ymax = y + sd(x))  
  
}


spore_data <- tibble(
  fraction_alive = abs(c(
    rnorm(6, 1e-1, 7e-2),
    rnorm(3, 1e-4, 7e-5), rnorm(3, 1e-3, 7e-4),
    rnorm(6, 1e-6, 7e-7),
    rnorm(6, 1e-8, 7e-9)
  )),
  heat = rep(rep(c("before heat", "after heat"), each = 3), 4),
  treatment = rep(c("A", "B", "C", "D"), each = 6)
)

spore_data %>% 
  mutate(log_fraction_alive = log10(fraction_alive),
         heat = factor(heat, levels = c("before heat", "after heat"))) %>%
  ggplot(aes(x = treatment, fill = heat, y = log_fraction_alive)) +
  stat_summary(geom = "col", fun.data = mean_sd, position = position_dodge(),
               color = "black", linewidth = 0.25) +
  stat_summary(geom = "errorbar", fun.data = mean_sd,
               position = position_dodge(width = 0.9), width = 0.3) +
  geom_hline(yintercept = c(0, -8), linewidth = c(0.5, 1),
             linetype = c("solid", "dashed")) +
  scale_fill_manual(name = NULL,
                    breaks = c("before heat", "after heat"),
                    values = c("#013062", "#D9ECF5")) +
  scale_x_discrete(name = NULL,
                   breaks = c("A", "B", "C", "D"),
                   labels = c("Val\nNis\nNoHP",
                            "Val\nNis\nmHP",
                            "Val\nNis\nvHP",
                            "TSB\nNis\nvHP\n80\u00B0C\n37\u00B0C\nvHP")) +
  scale_y_continuous(name = "log<sub>10</sub> (N/N<sub>0</sub>) [-]",
                     limits = c(-10, 0.5),
                     breaks = seq(0, -10, -1),
                     expand = c(0, 0)) +
  theme_classic() +
  theme(
    legend.position = "bottom",
    legend.direction = "vertical",
    legend.justification = "left",
    legend.text = element_text(face = "italic", size = 13),
    legend.background = element_rect(color = "black"),
    legend.margin = margin(2, 2, 2, 2),
    legend.key.height = unit(15, "pt"),
    legend.key.width = unit(30, "pt"),
    legend.box.spacing = unit(-40, "pt"),
    legend.box.margin = margin(l = 50),
    axis.text.x = element_text(size = 14, color = "black",
                               margin = margin(t = 8)),
    axis.text.y = element_text(size = 13, color = "black"),
    axis.ticks.y = element_blank(),
    axis.title.y = element_markdown(size = 14),
    axis.ticks.length.x = unit(-5, "pt"),
    plot.margin = margin(t = 5, r = 0, b = 8, l = 20),
    panel.grid.major.y = element_line(color = "gray", linewidth = 0.25)
  )
  
ggsave("negative_bars.png", width = 3.7, height = 5.2)
```