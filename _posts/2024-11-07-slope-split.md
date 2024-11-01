---
layout: post
title: "Making a labelled slope plot with the help of R's ggrepel and ggplot2 (CC313)"
blurb: "Visualizing a paired data set"
author: "PD Schloss"
date: 2024-11-07 8:00
comments: false
youtube: a-BCUcYwZOA
start_hash: 
end_hash: 
---

Pat uses ggplot2 and ggrepel to generate a labelled slope plot based on a similar plot showing survey sentiments for two groups of people. He customizes the appearance using pivot_longer, group_by, mutate, slice_max, geom_line, geom_vline, annotate, geom_text_repel, scale_x_discrete, scale_color_gradient2, coord_cartesian, labs, and theme. The newsletter describing how I would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/check-out-this-scatter-plot-with-annotation-in-swedish).

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)
library(ggrepel)
library(ggtext)

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
  pivot_longer(cols = c(farmer, non_farmer),
               names_to = "population", values_to = "rate") %>%
  mutate(population = factor(population,
                             levels = c("non_farmer", "farmer"))) %>%
  group_by(value) %>%
  mutate(diff = rate[1] - rate[2])

subset <- survey %>% 
  group_by(value) %>%
  slice_max(rate)

subset_farmer <- subset %>%
  filter(population == "farmer")

subset_non_farmer <- subset %>%
  filter(population == "non_farmer")

survey %>%
  ggplot(aes(x = population, y = rate, group = value, label = value,
             color = diff)) +
  geom_line(show.legend = FALSE) +
  geom_vline(xintercept = 0.85) +
  annotate("text", x = 0.78, y = c(0, 20, 40, 60),
           label = c("0%", "20%", "40%", "60%"), size = 3.5) +
  annotate("text", x = 0.84, y = 63, label = "Favorability", hjust = 1) + 
  geom_text_repel(data = subset_farmer, direction = "y",
                  nudge_x = 0.5, hjust = 1, size = 3, segment.color = "gray",
                  color = "black") + 
  geom_text_repel(data = subset_non_farmer, direction = "y",
                  nudge_x = -0.75, hjust = 0, size = 3, segment.color = "gray",
                  color = "black") + 
  scale_x_discrete(breaks = c("non_farmer", "farmer"),
                   labels = c("Non-farmer", "Farmer")) +
  scale_color_gradient2(low = "blue", high = "red", mid = "gray") +
  coord_cartesian(clip = 'off', ylim = c(0, 60)) +
  labs(x = NULL,
       y = "Percent favorability",
       title = "The main difference in favorability between farmers and non-farmers is there opinion of bicycling and walkability") +
  theme_classic() +
  theme(
    plot.title = element_textbox_simple(face = "bold", size = 20, margin = margin(b = 10)),
    axis.ticks = element_blank(),
    axis.line = element_blank(),
    axis.title = element_blank(),
    axis.text.y = element_blank(),
    axis.text.x = element_text(face = "bold")
  )

ggsave("slope_plot.png", width = 6, height = 5)
```