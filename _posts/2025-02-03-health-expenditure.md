---
layout: post
title: "Using ggplot2 to visualize relationship between life expectancy and health care spending in R (CC338)"
blurb: "Bubble plots with ggplot2"
author: "PD Schloss"
date: 2025-01-30 8:00
comments: false
youtube: mGLIlEopPFY
start_hash: 
end_hash: 
---

Pat recreates a figure from The Times showing the change in life expectancy by the amount of health care spending for seven countries using tools from R's dplyr, ggplot2, showtext, and ggtext. The functions he uses from these packages include aes, annotate, arrange, coord_cartesian, element_blank, element_line, element_markdown, element_rect, element_text, factor, format, geom_point, geom_text, ggplot, ggsave, labs, margin, mutate, read_tsv, scale_color_manual, scale_fill_manual, scale_size_continuous, scale_x_continuous, scale_y_continuous, select, seq, theme, and tribble. The original Times news article is [available here](https://www.thetimes.com/comment/columnists/article/we-keep-pumping-money-into-the-nhs-is-it-good-value-blq8bxc39). Pat's newsletter describing how he would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/representing-time-in-a-bubble-plot-with-ggplot2). The csv file can be [downloaded here](https://datawrapper.dwcdn.net/Bxhol/9/dataset.csv). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)
library(showtext)
library(ggtext)

font_add_google("Roboto", "roboto")
showtext_auto()
showtext_opts(dpi = 300)

country_labels <- tribble(
  ~country, ~year, ~spend, ~life_expectancy, ~xnudge, ~ynudge,
  "France", 2023,  5014,   83.3,             500,       0,
  "Germany",2023,  5971,   81.4,             600,       0,
  "UK",     2023,  4444,   81.3,             350,       0,
  "Italy",  2023,  3249,   83.7,             0,       0.55,
  "Canada", 2023,  5307,   82.6,             550,       0,
  "Japan",  2023,  4874,   84.7,             500,       0,
  "US",     2023, 10827,   79.3,             0,       0.5
)

read_tsv("https://datawrapper.dwcdn.net/Bxhol/9/dataset.csv") %>%
  select(country, year, spend,
         life_expectancy = le) %>%
  mutate(this_year = year == 2023,
         my_size = year %% 2000,
         my_size = if_else(year == 2023, 40, my_size + 1),
         country = factor(country,
                          levels = c("Canada", "France", "Germany", "Japan", 
                                     "Italy", "UK", "US"))) %>%
  arrange(country, year) %>%
  ggplot(aes(x = spend, y = life_expectancy,
             color = this_year, fill = country, size = my_size)) +
  geom_point(shape = 21, stroke = 0.25) +
  geom_text(data = country_labels,
            aes(x = spend, y = life_expectancy, label = country),
            nudge_x = country_labels$xnudge,
            nudge_y = country_labels$ynudge,
            family = "roboto", size = 9, size.unit = "pt", 
            inherit.aes = FALSE, color = "gray50") +
  annotate(geom = "rect",
           xmin = 1900, xmax = 2300, ymin = 86.4, ymax = 86.95,
           fill = "#E94F55") +
  annotate(geom = "text",
           x = c(2150, 11300),
           y = c(85.8, 77.5),
           hjust = c(0, 1), 
           label = c("Life expectancy", "Per-capita\nspend"),
           size = 9, size.unit = "pt", color = "gray50",
           lineheight = 0.9, 
           fontface = "bold", family = "roboto") +
  annotate(geom = "text",
           x = c(4300, 6400),
           y = c(77.85, 81),
           label = c(2000, 2023),
           # fontface = "bold", 
           color = "#F5C55E", family = "roboto", size = 9, size.unit = "pt") +
  scale_color_manual(breaks = c(FALSE, TRUE),
                     values = c("white", "black")) +
  scale_fill_manual(breaks = c("US", "France", "Italy", "Germany",
                               "UK", "Canada", "Japan"),
                    values = c("#4076A4", "#80B1E2", "#61A961", "#F5C55E",
                               "#E94F55", "#FFAEA9", "#DACFC0")) +
  scale_size_continuous(range = c(1, 6)) +
  scale_x_continuous(breaks = seq(3000, 11000, 1000),
                     labels = c(format(seq(3000, 10000, 1000),
                                       big.mark = ",", trim = TRUE),
                                "$11,000")) +
  scale_y_continuous() +
  coord_cartesian(xlim = c(2100, 11300),
                  ylim = c(77, 86), expand = FALSE, clip = "off") +
  labs(title = "Value for money",
       subtitle = "How life expectancy and per-capita healthcare spend have changed since 2000.<br><span style='color:white'>**UK**</span> spending is rising, but life expectancy has stalled",
       caption = "In US Dollars, adjusted for purchasing power and inflation. Excludes 2020-22.<br><span style='color:gray'>**Chart: Tom Calver | The Times and The Sunday Times**</span>"
       ) +
  theme(
    text = element_text(family = "roboto"),
    plot.title = element_text(face = "bold", size = 13,
                              margin = margin(l = 8, b = 5)),
    plot.subtitle = element_markdown(lineheight = 1.3,
                                     margin = margin(t = 3, l = 8, b = 11)),
    plot.title.position = "plot",
    plot.caption.position = "plot",
    plot.caption = element_markdown(hjust = 0, size = 8, lineheight = 1.5,
                                    margin = margin(t = 12, l = 8)),
    legend.position = "none",
    axis.text = element_text(color = "black"),
    axis.ticks = element_blank(),
    axis.line = element_line(linewidth = 0.25),
    axis.title = element_blank(),
    panel.grid.minor = element_blank(),
    panel.grid.major = element_line(linewidth = 0.25, color = "gray"),
    panel.background = element_rect(fill = "white")
  )

ggsave("health-expenditure.png", width = 6, height = 4.66)
```
