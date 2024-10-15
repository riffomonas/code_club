---
layout: post
title: "Customizing axes in a 1D scatter plot using the ggplot2 package in R (CC305)"
blurb: "Recreating a published scatter plot originally made in Prism with R"
author: "PD Schloss"
date: 2024-10-14 9:00
comments: false
youtube: DVTp6BO-pkM
start_hash: 
end_hash: 
---

Pat recreates a scatter plot that has a missing x-axis using the geom_point function from {ggplot2} in R using geom_point. He customizes the appearance to match the original figure using the `geom_vline`, `geom_hline`, `scale_color_manual`, `scale_x_continuous`, `scale_y_continuous`, `labs` and `theme` functions. You can find the code I develop in this episode at https://www.riffomonas.org/code_club/2024-10-14-scatter-plot.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```
library(tidyverse)
set.seed(19760620)

sim_data <- tibble(
  compound = 1:527,
  percent_inhibition = sample(
    c(rnorm(n = 464, mean = 0, sd = 30),
      runif(63, 91, 100))
    )
  ) %>%
  mutate(strong = percent_inhibition >= 90)

sim_data %>%
  ggplot(aes(x = compound, y = percent_inhibition, color = strong)) +
  geom_vline(xintercept = 1, linewidth = 1) +
  geom_hline(yintercept = c(0, 90),
             linetype = c("solid", "dashed"),
             linewidth = c(1, 0.5)) +
  geom_point(show.legend = FALSE) +
  scale_color_manual(
    breaks = c(FALSE, TRUE),
    values = c("seagreen", "maroon") # "gold4" was actual greenish color
  ) +
  coord_cartesian(clip = "off") +
  scale_x_continuous(expand = c(0,0)) +
  scale_y_continuous(limits = c(-101, 101),
                     breaks = seq(-100, 100, by = 50),
                     expand = c(0,0)) +
  labs(x = NULL,
       y = "Inhibition of Bacterial Growth %") +
  theme_classic() +
  theme(
    axis.line.x = element_blank(),
    axis.ticks.x = element_blank(),
    axis.text.x = element_blank(),
    axis.line.y = element_blank(),
    axis.ticks.y = element_line(linewidth = 1),
    axis.ticks.length.y = unit(5, "pt"),
    axis.text.y = element_text(face = "bold"),
    axis.title.y = element_text(face = "bold")
  )
```