---
layout: post
title: "Using ggplot2 and R to recreate a data journalist's figure about Michigan wages (CC321)"
blurb: "Implementing some cool effects"
author: "PD Schloss"
date: 2024-12-02 8:00
comments: false
youtube: B9FaffIOAVE
start_hash: 
end_hash: 
---

Pat recreates a journalist's figure from Bridge Michigan about the increase in Michigan wages using ggplot2. He'll show how to you can use ggplot2 functions to make a barplot with a customized x and y-axis, legend, and titles. He used geom_col, geom_text, annotate, scale_fill_manual, coord_cartesian, labs, and theme functions from R's ggplot2. The newsletter describing how he would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/reverse-engineering-a-dilution-series-of-box-plots).

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)
library(ggtext)

mi_wages <- tibble(
  year = as.character(c(2017, 2018, 2019, 2021, 2022, 2023)),
  earnings = c(46709, 47918, 48721, 52459, 55432, 57354),
  pretty = scales::label_currency()(earnings)
)

mi_wages %>%
  ggplot(aes(x = year, y = earnings, fill = "Median earnings",
             label = pretty)) +
  geom_col(width = 0.6, key_glyph = "point") +
  geom_text(y = 1000, angle = 90, hjust = 0, color = "white",
            size = 14, size.unit = "pt") +
  annotate("segment", x = seq(0.5, 6.5, 1), xend = seq(0.5, 6.5, 1),
           y = 0, yend = -4000) +
  annotate("segment", x = -1, xend = 6.6, y = -4500, yend = -4500) +
  annotate("text", x = -0.6, y = c(0, 20000, 40000, 60000) + 1000,
           label = c(0, "$20,000", "$40,000", "$60,000"),
           hjust = 0, vjust = 0, fontface = "bold",
           size = 14, size.unit = "pt") +
  scale_fill_manual(values = "darkred",
                    guide = guide_legend(override.aes = list(shape = 21,
                                                             size = 6))) +
  coord_cartesian(ylim = c(0, 61000),
                  xlim = c(-0.6, 6.6), clip = "off", expand = FALSE) +
  labs(x = NULL, y = "Med. earnings, full time workers",
       fill = NULL, 
       title = "Michigan wages increase",
       subtitle = "Median earnings of full-time workers rose from $46,700 in
                  2017 to more than $57,350 last year. But inflation erased
                  the buying power of those gains for most workers. But 
                  adjusted for inflation, median earnings were down 1.7%.",
       caption = "Source: U.S. Census, Bureau of Labor Statistics") +
  theme_classic() +
  theme(legend.position = "top",
        plot.title.position = "plot",
        plot.title = element_textbox_simple(face = "bold", size = 35,
                                            family = "serif", lineheight = 1),
        plot.subtitle = element_textbox_simple(size = 18, lineheight = 1,
                                               margin = margin(t = 8)),
        plot.caption.position = "plot",
        plot.caption = element_text(hjust = 0, face = "bold", size = 14,
                                    margin = margin(t = 10)),
        legend.text = element_text(face = "bold", size = 14),
        axis.ticks.x = element_blank(),
        axis.text.x = element_text(size = 15, face = "bold", color = "black"),
        axis.title.y = element_text(size = 13, face = "bold"),
        panel.grid.major.y = element_line(color = "black", linewidth = 0.25),
        axis.text.y = element_blank(),
        axis.ticks.y = element_blank(),
        axis.line.y = element_blank()
        )

ggsave("wages.png", width = 4.88, height = 8)
```