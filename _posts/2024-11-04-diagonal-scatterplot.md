---
layout: post
title: "Making a labelled scatter plot with the help of R's ggrepel and ggplot2 (CC312)"
blurb: "What do you value most about rural areas?"
author: "PD Schloss"
date: 2024-11-04 8:00
comments: false
youtube: 9wEW8DlIo_k
start_hash: 
end_hash: 
---

Pat uses ggplot2 and ggrepel to generate a labelled scatter plot based on a similar plot showing survey sentiments for two groups of people. He customizes the appearance using geom_point, geom_abline, geom_text, geom_text_repel, coord_equal, scale_x_continuous, scale_y_continuous, scale_color_viridis_c, labs, and theme. The newsletter describing how I would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/check-out-this-scatter-plot-with-annotation-in-swedish).

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)
library(ggrepel)

set.seed(19760620)

survey <- tibble(
    value = c("Happiness", "Cleanliness", "Peace and quiet", "Landscape",
              "Speed of life", "Biodiversity", "Recreation", "Aesthetics",
              "Cultural value", "Historical value", "Cost of living",
              "Population density", "Openness", "Tourism", "Smells",
              "Commerce", "Education", "Farm stands", "Bicycling",
              "Walkability"),
    clean = c(seq(1, 30, length = 18), 40, 50),
    farmer = abs(clean + runif(20, min = -5, max = 5)),
    non_farmer = abs(clean + runif(20, min = -5, max = 5))) %>%
  select(-clean) %>%
  mutate(diff = abs(farmer - non_farmer),
         pretty_label = if_else(diff > 3, value, ""))


survey %>%
  ggplot(aes(x = non_farmer, y = farmer, label = pretty_label, color = diff)) +
  geom_abline(slope = 1, intercept = 0, color = "gray",
              linetype = "dashed", linewidth = 0.25) +
  geom_point() +
  geom_text_repel(size = 3, min.segment.length = 0,
                  box.padding = 0.5,
                  point.padding = unit(5, "pt")) +
  coord_equal(xlim = c(0, 60), ylim = c(0, 60)) +
  scale_x_continuous(labels = scales::percent_format(scale = 1)) +
  scale_y_continuous(labels = scales::percent_format(scale = 1)) +
  scale_color_viridis_c(labels = scales::percent_format(scale = 1)) +
  labs(x = "Not farmer", y = "Farmer",
       title = "Values of Farmers differ little from Non-farmers",
       color = "Frequency\ndifference") +
  theme(panel.grid.minor = element_blank(),
        panel.grid.major = element_line(color = "gray", linewidth = 0.1),
        panel.background = element_rect(fill = "#F5FFF6"),
        axis.ticks = element_blank(),
        axis.text = element_text(face = "bold", color = "black"),
        axis.text.x = element_text(angle = 45),
        axis.title = element_text(face = "bold"),
        plot.title = element_text(face = "bold"),
        plot.title.position = "plot",
        legend.position = "top",
        legend.justification = "left",
        legend.title = element_text(size = 6, face = "bold", vjust = 1),
        legend.key.height = unit(10, "pt"),
        legend.text = element_text(size = 5, angle = -45, hjust = 0,
                                   margin = margin(t = 3)),
        legend.box.spacing = unit(0, "pt")
  )

ggsave("diagonal_scatter.png", width = 5, height = 5)
```