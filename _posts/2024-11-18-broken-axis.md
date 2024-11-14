---
layout: post
title: "Creating a broken x-axis using only the ggplot2 package in R (CC316)"
blurb: "Breaking axes"
author: "PD Schloss"
date: 2024-11-18 8:00
comments: false
youtube: BY1vY5c8eX4
start_hash: 
end_hash: 
---

Pat shows how to use annotate() and other functions from ggplot2 to create a break in the x-axis of a dotplot with error bars that were generated using stat_summary(). He customizes the appearance using bind_rows, factor, scale_color_manual, scale_fill_manual, coord_cartesian, scale_x_discrete, labs, and theme. The newsletter describing how I would go about generating the figure can be found [here](https://shop.riffomonas.org/posts/now-that-i-know-how-to-use-annotate-i-can-see-uses-for-it-everywhere).

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)
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
  bind_rows(tibble(week = 5,
                   condition = c("biological", "technical", "random"),
                   spearman = -2)) %>%
  mutate(condition = factor(condition,
                            levels = c("biological", "technical", "random")),
         week = as.character(week)) %>%
  ggplot(aes(x = week, y = spearman, color = condition, fill = condition)) +
  stat_summary(geom = "errorbar", fun.data = median_hilow, 
               position = position_dodge(width = 0.5),
               width = 0.4, show.legend = FALSE) +
  stat_summary(geom = "point", fun.data = median_hilow,
               shape = 21, color = "black", size = 3,
               position = position_dodge(width = 0.5)) +
  annotate("segment", x = c(0.75, 6.25), xend = c(5.75, 8.25),
           y = -0.6, yend = -0.6,
           color = "black", linewidth = 0.5) +
  annotate("segment", x = c(1, 2, 3, 4, 5, 7, 8), xend = c(1, 2, 3, 4, 5, 7, 8),
           y = -0.6, yend = -0.65,
           color = "black", linewidth = 0.5) +
  annotate("segment",
           x = c(5.85, 6.35), xend = c(5.65, 6.15),
           y = c(-0.55, -0.55), yend = c(-0.65, -0.65)) +
  scale_color_manual(breaks = c("biological", "technical", "random"),
                     values = c("dodgerblue", "darkgray", "black"),
                     labels = c("biological variability",
                                "technical variability", 
                                "random permutation")) +
  scale_fill_manual(breaks = c("biological", "technical", "random"),
                    values = c("dodgerblue", "darkgray", "darkred"),
                    labels = c("biological variability",
                               "technical variability", 
                               "random permutation")) +
  coord_cartesian(ylim = c(-0.6, 1.0), expand = FALSE, clip = "off") +
  scale_x_discrete(breaks = c(0, 1, 2, 3, 4, 5, 8, 9),
                   labels = c(0, 1, 2, 3, 4, "", 8, 9)) +
  labs(color = NULL, fill = NULL,
       x = "time (weeks)", y = "Spearman R") +
  theme_classic() +
  theme(legend.position = "bottom",
        legend.justification = "left",
        legend.key.size = unit(4, "pt"),
        legend.key.spacing.x = unit(12, "pt"),
        axis.line.x = element_blank(),
        axis.ticks.x = element_blank(),
        axis.ticks.length.y = unit(7, "pt"),
        axis.text.x = element_text(margin = margin(t = 7)))
  

ggsave("broken_x_axis.png", width = 6, height = 4)
```