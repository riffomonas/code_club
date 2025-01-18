---
layout: post
title: "Positron or RStudio? Using Positron for the first time to create a scatter plot with the ggplot2 R package (CC333)"
blurb: "Or Virtual Studio Code?"
author: "PD Schloss"
date: 2025-01-16 8:00
comments: false
youtube: 2kKtFVmpels
start_hash: 
end_hash: 
---

Pat installs and uses Positron for the first time to recreate a visualization generated from the gapminder dataset. He compares using Positron to RStudio as he builds this figure. To recreate this visualization, Pat uses geom_point, geom_text, scale_fill_manual, scale_x_log10, coord_cartesian, labs, and theme. If you have a figure that you would like to see Pat discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)
library(gapminder)

gm_2007 <- gapminder %>%
  filter(year == 2007)

gm_2007_labels <- gm_2007 %>%
  filter(country %in% c("United States", "India", "China")) %>%
  mutate(
    country = if_else(country == "United States", "USA", country),
    gdpPercap = c(4000, 2200, 70000),
    lifeExp = c(77, 69, 78)
  )

gm_2007 %>%
  ggplot(aes(x = gdpPercap, y = lifeExp, size = pop, fill = continent)) +
  geom_point(shape = 21, color = "white", show.legend = FALSE) +
  geom_text(
    data = gm_2007_labels,
    mapping = aes(label = country),
    size = 12, size.unit = "pt"
  ) +
  scale_fill_manual(
    breaks = c("Africa", "Americas", "Asia", "Europe", "Oceania"), 
    values = c("forestgreen", "red", "blue", "orange", "purple")
  ) +
  scale_x_log10(
    breaks = c(1e3, 1e4, 1e5),
    limits = c(200, 1e5),
    expand = c(0, 0),
    labels = c("1k", "10k", "100k")
  ) +
  coord_cartesian(clip = "off") +
  labs(
    x = "GDP per Capita [in USD]",
    y = "Life Expectancy [in years]",
    title = "Global Development in 2007") +
  theme(
      plot.title = element_text(hjust = 0.5),
      plot.margin = margin(t = 5, r = 10, b = 5, l = 5),
      panel.grid.minor = element_blank(),
      axis.ticks = element_blank()
    )
    
ggsave("gapminder_2007.png", width = 5, height = 3)
```
