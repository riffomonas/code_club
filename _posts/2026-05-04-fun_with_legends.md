---
layout: post
title: "How to give legends organization in R with ggplot2 (CC421)"
blurb: "Fun with legends"
author: "PD Schloss"
date: 2026-05-04 12:00
comments: false
youtube: F78cYOjZGJ4
start_hash: 
end_hash: 
---

Inspired by the legends in several recently found figures published in Nature and Nature Microbiology, Pat shows how to create legend subsections and assign multiple plotting symbols and colors to the same category. He uses the penguins dataset from base R along with tools from ggplot2 and dplyr. The functions he uses in this episode included aes, drop_na, element_text, factor, geom_point, ggplot, ggsave, guide_legend, library, list, margin, mutate, paste, scale_color_manual, scale_fill_manual, scale_shape_manual, theme, and unit. If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```R
library(tidyverse)

# basic legend

penguins %>%
  drop_na() %>%
  mutate(species_island = paste(species, island),
         species_island = factor(species_island,
                                 levels = c("Adelie Biscoe", "Gentoo Biscoe", 
                                            "Adelie Dream", "Chinstrap Dream",
                                            "Adelie Torgersen"))) %>%
  ggplot(aes(x = bill_dep, y = bill_len, color = species_island)) +
  geom_point() +
  scale_color_manual(
    name = NULL,
    values = c("Adelie Torgersen" = "black", "Adelie Biscoe" = "blue",
               "Adelie Dream" = "red", "Gentoo Biscoe" = "orange",
               "Chinstrap Dream" = "purple")
  ) +
  theme(
    legend.text = element_text(size = 10, margin = margin(l = 3)),
    legend.key.size = unit(12, "pt"),
    legend.key = element_blank()
  )

ggsave("basic.png", width = 6, height = 3)


# legend with subsections

penguins %>%
  drop_na() %>%
  mutate(species_island = paste(species, island)) %>%
  ggplot(aes(x = bill_dep, y = bill_len, color = species_island,
             fill = species_island, shape = species_island)) +
  geom_point() +
  scale_color_manual(
    name = "Biscoe",
    values = c("Adelie Torgersen" = "black", "Adelie Biscoe" = "blue",
               "Adelie Dream" = "red", "Gentoo Biscoe" = "orange",
               "Chinstrap Dream" = "purple"), 
    breaks = c("Adelie Biscoe", "Gentoo Biscoe"),
    labels = c("Adelie", "Gentoo")
  ) +
  scale_fill_manual(
    name = "Dream",
    values = c("Adelie Torgersen" = "black", "Adelie Biscoe" = "blue",
               "Adelie Dream" = "red", "Gentoo Biscoe" = "orange",
               "Chinstrap Dream" = "purple"), 
    breaks = c("Adelie Dream", "Chinstrap Dream"),
    labels = c("Adelie", "Chinstrap"),
    guide = guide_legend(
      override.aes = list(color = c("Adelie Dream" = "red",
                                    "Chinstrap Dream" = "purple"))
    )
  ) +
  scale_shape_manual(
    name = "Torgersen",
    values = c("Adelie Torgersen" = 19, "Adelie Biscoe" = 19,
               "Adelie Dream" = 19, "Gentoo Biscoe" = 19,
               "Chinstrap Dream" = 19), 
    breaks = c("Adelie Torgersen"),
    labels = c("Adelie"),
    guide = guide_legend(
      override.aes = list(color = c("Adelie Torgersen" = "black"))
      ) 
  ) +
  theme(
    legend.key.size = unit(12, "pt"),
    legend.key = element_blank()
  )
  
ggsave("subsections.png", width = 6, height = 3)


# legend with multiple plotting symbols per category

penguins %>%
  drop_na() %>%
  mutate(species_island = paste(species, island)) %>%
  ggplot(aes(x = bill_dep, y = bill_len, color = species_island,
             fill = species_island, shape = species_island)) +
  geom_point() +
  scale_color_manual(
    name = "Biscoe",
    values = c("Adelie Torgersen" = "black", "Adelie Biscoe" = "blue",
               "Adelie Dream" = "red", "Gentoo Biscoe" = "orange",
               "Chinstrap Dream" = "purple"), 
    breaks = c("Adelie Biscoe", "Gentoo Biscoe"),
    labels = c("Adelie", "Gentoo"),
    guide = guide_legend(
      order = 1,
      override.aes = list(size = 3))
  ) +
  scale_fill_manual(
    name = "Dream",
    values = c("Adelie Torgersen" = "black", "Adelie Biscoe" = "blue",
               "Adelie Dream" = "red", "Gentoo Biscoe" = "orange",
               "Chinstrap Dream" = "purple"), 
    breaks = c("Adelie Dream", "Chinstrap Dream"),
    labels = c("Adelie", "Chinstrap"),
    guide = guide_legend(
      order = 2,
      override.aes = list(color = c("Adelie Dream" = "red",
                                    "Chinstrap Dream" = "purple"),
                          size = 3)
    )
  ) +
  scale_shape_manual(
    name = "Torgersen",
    values = c("Adelie Torgersen" = 19, "Adelie Biscoe" = 19,
               "Adelie Dream" = 19, "Gentoo Biscoe" = 19,
               "Chinstrap Dream" = 19), 
    breaks = c("Adelie Dream", "Adelie Torgersen"),
    labels = c("Adelie", "Adelie"),
    guide = guide_legend(
      order = 3, 
      override.aes = list(color = c("Adelie Dream" = NA, 
                                    "Adelie Torgersen" = "black"),
                          size = 3)
    ) 
  ) +
  theme(
    legend.text = element_blank(),
    legend.title = element_text(margin = margin(l = 3), size = 10),
    legend.direction = "horizontal",
    legend.title.position = "right",
    legend.spacing.y = unit(0, "pt"),
    legend.margin = margin(0, 0, 0, 0),
    legend.key.spacing = unit(0, "pt"),
    legend.key.size = unit(12, "pt"),
    legend.key = element_blank()
  )

ggsave("multisymbol_groups.png", width = 6, height = 3)
```