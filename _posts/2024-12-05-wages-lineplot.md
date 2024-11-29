---
layout: post
title: "Data journalism makeover with ggplot2 in R to make labelled line plot with google fonts (CC322)"
blurb: "Fun with google fonts"
author: "PD Schloss"
date: 2024-12-05 8:00
comments: false
youtube: kToIwB2mW5I
start_hash: 
end_hash: 
---

Pat makes over a journalist's figure from Bridge Michigan about the increase in Michigan wages using ggplot2 and showtext. He'll show how to you can use ggplot2 functions to make a line plot with labels in place of a legend, customized labels and axes, and fonts imported from google fonts. He used font_add_google, showtext_auto, and showtext_opts from showtext and geom_line, annotate, labs, scale_color_manual, scale_x_continuous, scale_y_continuous, and theme from ggplot2. The newsletter describing how he would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/reverse-engineering-a-dilution-series-of-box-plots).

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)
library(ggtext)
library(showtext)

font_add_google("Playfair Display", "playfair_display")
font_add_google("Open Sans", "open_sans")

showtext_auto()
showtext_opts(dpi = 300)

# Data...
# https://data.census.gov/table/ACSST1Y2017.S2001?g=040XX00US26
# 2020 is based on 5 year estimate data

# Inflation adjustment to 2017 data (used January for all years)...
# https://data.bls.gov/cgi-bin/cpicalc.pl

mi_wages <- tibble(
  year = c(2017, 2018, 2019, 2020, 2021, 2022, 2023),
  earnings = c(46709, 47918, 48721, 49931, 52459, 55432, 57354),
  adjusted_to_2017 = c(46709, 46946, 46050, 46049, 48700, 47879, 46555)
)

mi_wages %>%
  pivot_longer(cols = -year, names_to = "type", values_to = "dollars") %>%
  ggplot(aes(x = year, y = dollars, color = type)) +
  geom_line(show.legend = FALSE, linewidth = 2, lineend = "round") +
  annotate("text",
           x = c(2022, 2020.25), y = c(56000, 49300),
           label = c("Actual wages", "Adjusted to 2017"),
           hjust = c(1, 0),
           color = c("darkgreen", "black"),
           size = 14, size.unit = "pt",
           fontface = "bold", family = "open_sans") +
  labs(x = "Year", y = "Median earnings, full time workers",
       title = "Michigan wages increase",
       subtitle = "Median earnings of full-time workers rose from $46,700 in
                  2017 to more than $57,350 last year. But inflation erased
                  the buying power of those gains for most workers. But 
                  adjusted for inflation, median earnings were down 1.7%.",
       caption = "Source: U.S. Census, Bureau of Labor Statistics") +
  scale_color_manual(breaks = c("earnings", "adjusted_to_2017"),
                    values = c("darkgreen", "black")) +
  scale_x_continuous(breaks = c(2017, 2019, 2021, 2023)) +
  scale_y_continuous(limits = c(45000, 60000),
                     labels = scales::label_currency()) +
  theme_classic() +
  theme(
    text = element_text(family = "open_sans"),
    plot.title.position = "plot",
    plot.caption.position = "plot",
    plot.title = element_textbox_simple(face = "bold", size = 35,
                                        lineheight = 1,
                                        family = "playfair_display"),
    plot.subtitle = element_textbox_simple(size = 18, lineheight = 1,
                                           margin = margin(t = 10, b = 10)),
    plot.caption = element_text(hjust = 0),
    axis.text = element_text(size = 14, color = "black", face = "bold"),
    axis.title = element_text(size = 15, face = "bold"),
    axis.ticks = element_blank(),
    panel.grid.major.y = element_line()
  )

ggsave("wages.png", width = 4.88, height = 8)
```