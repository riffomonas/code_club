---
layout: post
title: "Recreating a "U-turn" plot from the New York Times in R with ggplot2 (CC396)"
blurb: "Like a barbell plot but reverses itself"
author: "PD Schloss"
date: 2026-01-28 12:00
comments: false
youtube: Nyat8TSDvUA
start_hash: 
end_hash: 
---

In this livestream, Pat creates what he's calling a u-turn plot based on a visualization published by Nate Cohn in the New York Times. He created the figure using R, dplyr, ggplot2, ggtext, showtext, and other tools from the tidyverse. The functions he used from these packages included aes, annotate, arrow, bind_rows, coord_cartesian, cos, distinct, element_blank, element_line, element_text, element_textbox_simple, else, font_add_google, function, geom_path, geom_text, geom_vline, ggplot, ggsave, if, labs, library, map, margin, mutate, n, nest, scale_x_continuous, scale_y_continuous, select, seq, showtext_auto, showtext_opts, sin, theme, tibble, tribble, and unnest. The original NY Times article that included this figure can be [found here](https://www.nytimes.com/2026/01/22/upshot/trump-poll-analysis-times-siena.html). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```R
library(tidyverse)
library(ggtext)
library(showtext)

font_add_google("Domine", family = "domine")
font_add_google("Libre Franklin", family = "franklin_bold", bold.wt = 800)
font_add_google("Libre Franklin", family = "franklin", bold.wt = 500)
showtext_opts(dpi = 300)
showtext_auto()

generate_uturn_data <- function(levels,
                                radius = 1, stretch = 0.5, resolution = 20) {
  
  y_position <- c(radius, 0, -radius)
  
  top <- if(levels[1] <= levels[2]){
    # top right quadrant
    theta <- seq(pi/2, 0, length.out = resolution)
    bind_rows(
      tibble(x = levels[1], y = stretch),
      tibble(
        x = radius * cos(theta) - radius + levels[2],
        y = stretch * sin(theta)
      )
    )
  } else {
    # top left quadrant
    theta <- seq(pi/2, pi, length.out = resolution)
    bind_rows(
      tibble(x = levels[1], y = stretch),
      tibble(
        x = radius * cos(theta) + radius + levels[2],
        y = stretch * sin(theta)
        )
    )
  }
  
  bottom <- if(levels[3] <= levels[2]) {
    # bottom right quadrant
    theta <- seq(0, -pi/2, length.out = resolution)
    bind_rows(
      tibble(
        x = radius * cos(theta) - radius + levels[2],
        y = stretch * sin(theta)
      ),
      tibble(x = levels[3], y = -stretch)
    )
  } else {
    # bottom left quadrant
    theta <- seq(pi, 3 * pi / 2, length.out = resolution)
    bind_rows(
      tibble(
        x = radius * cos(theta) + radius + levels[2],
        y = stretch * sin(theta)
      ),
      tibble(x = levels[3], y = -stretch)
    )
  }
  
  bind_rows(top, bottom)
}


d <- tribble(
  ~group, ~year, ~approval,
  "Nonwhite", 2020, -38,
  "Nonwhite", 2024, -20,
  "Nonwhite", 2026, -45,
  "Age 18-29", 2020, -28,
  "Age 18-29", 2024, -8,
  "Age 18-29", 2026, -40,
  "Not midterm voters", 2020, -8,
  "Not midterm voters", 2024, 3,
  "Not midterm voters", 2026, -15,
  "All registered voters", 2020, -8,
  "All registered voters", 2024, -2,
  "All registered voters", 2026, -13,
  "Midterm voters", 2020, -8,
  "Midterm voters", 2024, -5,
  "Midterm voters", 2026, -10,
  "Age 65+", 2020, -8,
  "Age 65+", 2024, -3,
  "Age 65+", 2026, -5,
  "White", 2020, 7,
  "White", 2024, 6,
  "White", 2026, 5
) %>%
  nest(data = - group) %>%
  mutate(row_index = n():1) %>%
  mutate(uturn = map(data,
                     ~generate_uturn_data(.x$approval, stretch = 0.15))) %>%
  unnest(uturn) %>%
  mutate(y = row_index + y)

d %>%
  ggplot(aes(x = x, y = y, group = group)) +
  annotate(geom = "rect",
           xmin = -78, xmax = 18, ymin = 3.5, ymax = 4.5,
           color = NA, fill = "#F6F7F9") +
  geom_vline(xintercept = c(-50, -30, -10, 0, 10),
             linewidth = 0.2,
             color = c(rep("#d2d3d4", 3), "black", "#d2d3d4")) +
  geom_path(linewidth = 3, color = "#A1B3C4",
            linejoin = "mitre",
            arrow = arrow(length = unit(3, "pt"), angle = 30)) +
  geom_text(
    data = select(d, group, row_index) %>% distinct(),
    aes(label = group, y = row_index), x = -77, hjust = 0,
    family = "franklin", size = 12.5, size.unit = "pt", fontface = "bold") +
  coord_cartesian(
    xlim = c(-78, 12),
    ylim = c(0.5, 7.75),
    expand = FALSE, clip = "off",
  ) +
  scale_y_continuous(breaks = seq(1.5, 6.5, 1)) +
  scale_x_continuous(breaks = c(-50, -30, -10, 10),
                     labels = c("-50 pts.", -30, -10, "+10"),
                     position = "top") +
  labs(
    title = "How Support for Trump Has Changed",
    subtitle = "President Trump made gains with most groups between 2020 and 2024. But in the latest Times/Siena poll, those shifts have vanished with young, nonwhite and infrequent voters.",
    caption = "Source: New York Times/Siena University polls of registered voters in October 2020, late October 2024, September 2025 and January 2026. The 2025 and 2026 data are combined, and for comparability, they include only interviews completed without text messaging. <span style='font-size:6pt;'>By The New York Times</span>",
    x = NULL,
    y = NULL
  ) +
  annotate(geom = "text",
           x = c(-38-3, -20+3, -45-4),
           y = c( 7.15, 7, 6.85),
           label = c("'20", "'24", "'26"), family = "franklin",
           fontface = "bold") +
  annotate(geom = "text",
           x = c(-55, -77), y = c(8.5, 7.6),
           label = c("Net support", "Group"),
           hjust = 0, family = "franklin", size = 13, size.unit = "pt") +
  theme(
    text = element_text(family = "franklin"),
    panel.background = element_blank(),
    panel.grid.minor = element_blank(),
    panel.grid.major.y = element_line(linewidth = 0.2, color = "#d2d3d4"),
    panel.grid.major.x = element_blank(),
    plot.margin = margin(t = 12, r = 10, b = 3, l = 13),
    plot.title = element_text(face = "bold", size = 15, family = "franklin_bold"),
    plot.subtitle = element_textbox_simple(size = 13, color = "gray40",
                                           margin = margin(t = 0, b = 35)),
    plot.caption = element_textbox_simple(family = "domine",
                                          margin = margin(t = 10, b = 3)),
    axis.text.y = element_blank(),
    axis.text.x.top = element_text(size = 12.5, margin = margin(b = 6)),
    axis.ticks = element_blank()
  )

ggsave("nytimes_uturn_plot.png", width = 6, height = 6.34)
```