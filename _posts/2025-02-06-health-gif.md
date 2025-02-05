---
layout: post
title: "Using gganimate to animate changes in life expectancy and health care spending with R (CC339)"
blurb: "Animating bubble plots with gganimate"
author: "PD Schloss"
date: 2025-02-06 8:00
comments: false
youtube: qD1TlRdcFCk
start_hash: 
end_hash: 
---

Pat puts a twist on a figure from The Times by creating an animation of the change in life expectancy by the amount of health care spending for seven countries using tools from R's gganimate, dplyr, ggplot2, showtext, and ggtext. The functions he uses from these packages include aes, anim_save, animate, annotate, arrange, coord_cartesian, factor, font_add_google, geom_label, geom_point, geom_text, ggplot, ggsave, if_else, inner_join, labs, magick_renderer, mutate, read_tsv, scale_color_manual, scale_fill_manual, scale_x_continuous, scale_y_continuous, select, shadow_trail, showtext_auto, showtext_opts, theme, transition_manual, and tribble. The data and code from the previous episode that this one builds upon can be [found here](https://www.riffomonas.org/code_club/2025-02-03-health-expenditure). The original Times news article is [available here](https://www.thetimes.com/comment/columnists/article/we-keep-pumping-money-into-the-nhs-is-it-good-value-blq8bxc39). Pat's newsletter describing how he would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/representing-time-in-a-bubble-plot-with-ggplot2). The csv file can be [downloaded from here](https://datawrapper.dwcdn.net/Bxhol/9/dataset.csv). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)
library(showtext)
library(ggtext)
library(gganimate)

font_add_google("Roboto", "roboto")
showtext_auto()
showtext_opts(dpi = 300)

country_labels <- tribble(
  ~country, ~xnudge, ~ynudge,
  "France", 500,       0,
  "Germany",600,       0,
  "UK",     350,       0,
  "Italy",  0,       0.55,
  "Canada", 550,       0,
  "Japan",  500,       0,
  "US",     0,       0.5
)

data <- read_tsv("https://datawrapper.dwcdn.net/Bxhol/9/dataset.csv") %>%
  select(country, year, spend,
         life_expectancy = le) %>%
  mutate(country = factor(country,
                          levels = c("Canada", "France", "Germany", "Japan", 
                                     "Italy", "UK", "US"))) %>%
  arrange(country, year) %>%
  inner_join(country_labels, by = "country") %>%
  mutate(ynudge = if_else(country == "Canada" & year <= 2003,
                          0.3,
                          ynudge),
         ynudge = if_else(country == "Canada" & year > 2003 & year <= 2016,
                          -0.3,
                          ynudge),
         ynudge = if_else(country == "France" & year > 2003 & year <= 2016,
                          +0.2,
                          ynudge))



animation <- data %>%
  ggplot(aes(x = spend, y = life_expectancy, fill = country,
             label = country, color = country)) +
  geom_point(shape = 21, stroke = 0.25, size = 6, color = "black") +
  geom_text(nudge_x = data$xnudge,
            nudge_y = data$ynudge,
            family = "roboto", size = 9, size.unit = "pt", fontface = "bold") +
  geom_label(aes(label = year), x = 10000, y = 85,
             color = "black", fill = "white", label.size = unit(0, "mm"),
             size = 20, size.unit = "pt", family = "roboto",
             fontface = "bold") +
  annotate(geom = "rect",
           xmin = 1900, xmax = 2300, ymin = 86.5, ymax = 87.05,
           fill = "#E94F55") +
  annotate(geom = "text",
           x = c(2150, 11300),
           y = c(85.8, 76.5),
           hjust = c(0, 1), 
           label = c("Life expectancy", "Per-capita\nspend"),
           size = 9, size.unit = "pt", color = "gray50",
           lineheight = 0.9, 
           fontface = "bold", family = "roboto") +
  scale_color_manual(breaks = c("US", "France", "Italy", "Germany",
                               "UK", "Canada", "Japan"),
                    values = c("#4076A4", "#80B1E2", "#61A961", "#F5C55E",
                               "#E94F55", "#FFAEA9", "#DACFC0")) +
  scale_fill_manual(breaks = c("US", "France", "Italy", "Germany",
                               "UK", "Canada", "Japan"),
                    values = c("#4076A4", "#80B1E2", "#61A961", "#F5C55E",
                               "#E94F55", "#FFAEA9", "#DACFC0")) +
  scale_x_continuous(breaks = seq(3000, 11000, 1000),
                     labels = c(format(seq(3000, 10000, 1000),
                                       big.mark = ",", trim = TRUE),
                                "$11,000")) +
  scale_y_continuous() +
  coord_cartesian(xlim = c(2100, 11300),
                  ylim = c(76, 86), expand = FALSE, clip = "off") +
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
  ) +
  transition_manual(year) +
  shadow_trail(alpha = 0.25, size = 4, label = "")

animate(animation, width = 6, height = 4.66, units = "in", res = 300,
        nframes = 21, fps = 4, renderer = magick_renderer(loop = FALSE))

anim_save("health-expenditure.gif")
```
