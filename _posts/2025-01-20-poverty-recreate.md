---
layout: post
title: "Using ggplot2 to see how much has the rate of poverty changed in the US over the past 65 years (CC334)"
blurb: "National Poverty Awareness Month"
author: "PD Schloss"
date: 2025-01-16 8:00
comments: false
youtube: biPe61VD8oY
start_hash: 
end_hash: 
---

Pat recreates a figure showing the change in the number and percent of people in the US who live in poverty for the past 65 years in honor of National Poverty Awareness Month. He pulls this off using tools from ggplot2, dplyr, readxl, jsonlite, ggtext, and showtext. The functions he uses from these packages include font_add_google, showtext_auto, showtext_opts, select, filter, mutate, pivot_longer, inner_join, str_replace, case_when, row_number, paste, year, yday, read_excel, geom_rect, geom_hline, geom_line, geom_text, facet_wrap, labeller, labs, scale_color_manual, scale_fill_manual, scale_y_continuous, scale_x_continuous, coord_cartesian, guides, guide_axis, theme_classic, and theme. Pat's newsletter describing how he would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/tracking-trends-in-poverty-with-ggplot2). The recession data can be [found here](https://data.nber.org) and the poverty data can be [found here](https://www.census.gov/data/tables/time-series/demo/income-poverty/historical-poverty-people.html). The xlsx file can be [downloaded from here](https://www2.census.gov/programs-surveys/cps/tables/time-series/historical-poverty-people/hstpov2.xlsx). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

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
