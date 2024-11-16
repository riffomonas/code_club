---
layout: post
title: "Putting a break in the x-axis using facet_grid from ggplot2 R package (CC317)"
blurb: "Breaking axes"
author: "PD Schloss"
date: 2024-11-21 8:00
comments: false
youtube: z3cbJ8vpZjY
start_hash: 
end_hash: 
---

Pat shows how to use facet_grid() and other functions from ggplot2 to create a break in the x-axis of a dot plot with error bars that were generated using stat_summary(). He customizes the appearance using scale_x_continuous, scale_color_manual, labs, and theme. The newsletter describing how I would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/now-that-i-know-how-to-use-annotate-i-can-see-uses-for-it-everywhere).

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```
library(tidyverse)
library(ggtext)

set.seed(19760620)

e_data <- tibble(
  condition = c(rep("technical", 108 * 7),
                rep("random", 108 * 7),
                rep("biological", 18 * 6)),
  week = c(rep(c(0, 1, 2, 3, 4, 8, 9), each = 108),
           rep(c(0, 1, 2, 3, 4, 8, 9), each = 108),
           rep(c(1, 2, 3, 4, 8, 9), each = 18)
  ),
  spearman = c(
    #technical
    runif(108, 0.50, 0.90), runif(108, 0.85, 0.95), runif(108, 0.90, 0.97),
    runif(108, 0.50, 0.85), runif(108, 0.85, 0.95), runif(108, 0.93, 0.97),
    runif(108, 0.98, 1.00),
    
    #random
    runif(540, -0.4, 0.4), runif(216, -0.6, 0.6),
               
    #biological
    runif(18, 0.20, 0.65), runif(18, 0.55, 0.85), runif(18, 0.58, 0.88),
    runif(18, 0.58, 0.88), runif(18, 0.50, 1.00), runif(18, 0.25, 0.75)
  )
)


e_data %>%
  mutate(phase = if_else(week < 5, "early", "late"),
         condition = factor(condition,
                            levels = c("biological", "technical", "random"))) %>%
  ggplot(aes(x = week, y = spearman, color = condition)) +
  stat_summary(fun.data = median_hilow, 
               position = position_dodge(width = 0.5),
               key_glyph = "point") +
  facet_grid(~phase, scale = "free_x", space = "free_x") +
  scale_x_continuous(breaks = 0:9) +
  scale_color_manual(breaks = c("biological", "technical", "random"),
                     labels = c("biological variability",
                                "technical variability",
                                "random permutation"),
                     values = c("dodgerblue", "darkgray", "darkred"),
                     guide = guide_legend(override.aes = list(size = 2))) +
  labs(x = "Time (weeks)", y = "Spearman correlation<br>coefficient (&rho;)",
       color = NULL) +
  theme_classic() +
  theme(
    panel.background = element_rect(color = "black", linewidth = 1),
    axis.line = element_blank(),
    strip.text = element_blank(),
    axis.title.y = element_markdown(),
    legend.key = element_rect(color = "white"),
    legend.position = "bottom",
    legend.box.spacing = unit(0, "pt"),
    legend.key.width = unit(0, "pt"),
    legend.text = element_text(margin = margin(l = 6, r = 10))
  )

ggsave("faceted_x_axis.png", width = 6, height = 4)
```