---
layout: post
title: "Using ggplot2's facet_wrap to create three panelled figure in R (CC311)"
blurb: "Three different y-axes!"
author: "PD Schloss"
date: 2024-10-31 8:00
comments: false
youtube: wgq9853P9pg
start_hash: 
end_hash: 
---

Pat does a data viz makeover to convert a two panelled figure into a three panelled figure using facet_wrap from the ggplot2 R package. He customizes the appearance using pivot_longer, geom_boxplot, geom_segment, geom_text, geom_hline, scale_y_continuous, coord_cartesian, labs, and theme functions. The original figure was published in the [journal mBio](https://journals.asm.org/doi/10.1128/mbio.01966-24); Pat recreated Figure 1. A [previous newsletter](https://shop.riffomonas.org/posts/fun-with-boxplots) described how he would go about generating the figure.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)
library(ggtext)

set.seed(19760620)

coral_physiology <- tibble(
  fragment = 1:72,
  temperature = rep(c(25, 32), each = 36),
  density = c(rnorm(n = 36, mean = 32, sd = 10),
              rnorm(n = 36, mean = 5, sd = 1)),
  photosynthesis = abs(c(rnorm(n = 36, mean = 4.5, sd = 1),
                     rnorm(n = 36, mean = 2, sd = 1))),
  respiration = abs(c(rnorm(n = 36, mean = -2, sd = 0.75),
                  rnorm(n = 36, mean = -1, sd = 0.75)))
)

pretty_names <- c("Symbionts (x10<sup>5</sup> cm<sup>-2</sup>)",
                  "Photosynthesis<br>(&mu;mol O<sub>2</sub> cm<sup>-2</sup> h<sup>-1</sup>)",
                  "Respiration<br>(-&mu;mol O<sub>2</sub> cm<sup>-2</sup> h<sup>-1</sup>)")
names(pretty_names) <- c("density", "photosynthesis", "respiration")

sig_table <- tibble(
  name = c("density", "photosynthesis", "respiration"),
  x = 1.1, xend = 1.9,
  y = c(55, 8, 4),
  yend = y
)

point_table <- tibble(
  name = c("density", "photosynthesis", "respiration"),
  x = 1.5, xend = 1.9,
  y = c(55, 8, 4) * 1.02,
  label = "*"
)

coral_physiology %>%
  pivot_longer(cols = c(density, photosynthesis, respiration)) %>%
  ggplot(aes(x = as.character(temperature), y = value)) +
  geom_boxplot() +
  geom_segment(data = sig_table, aes(x = x, y = y, xend = xend, yend = yend)) +
  geom_text(data = point_table, aes(x = x, y = y, label = label),
            size = 5, fontface = "bold") +
  geom_hline(yintercept = 0) +
  facet_wrap(facets = ~name, scales = "free_y", nrow = 3,
             strip.position = "left",
             labeller = labeller(name = pretty_names)) +
  scale_y_continuous(limits = c(0, NA), expand = c(0,0)) +
  coord_cartesian(clip = "off") +
  labs(x = "Temperature (&deg;C)", y = NULL) +
  theme_classic() +
  theme(
    strip.placement = "outside",
    strip.background = element_blank(),
    strip.text = element_markdown(),
    axis.title.x = element_markdown(),
    panel.spacing = unit(15, units = "pt")
  )

ggsave("faceted-boxplot.png", width = 3, height = 5)
```