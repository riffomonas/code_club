---
layout: post
title: "Visualizing the US baby boom as a NY Times-styled heatmap (CC360)"
blurb: "Remixing two data visualizations"
author: "PD Schloss"
date: 2025-06-18 8:00
comments: false
youtube: K646B0ZdhfU
start_hash: 
end_hash: 
---

Pat revisualizes data from the human fertility database to depict the fertility explosion between 1946 and 1964 as a heatmap that is styled like a figure you would typically find in the New York Times. The plot was generated using functions from the tidyverse including the dplyr, ggplot2, showtext, and ggtext packages. The functions he uses from these packages include aes, annotate, arrow, as.numeric, coord_equal, element_blank, element_line, element_markdown, element_text, filter, font_add_google, geom_tile, geom_vline, ggplot, ggsave, labs, library, margin, mutate, read_table, rename_all, scale_fill_viridis_c, scale_x_continuous, scale_y_continuous, seq, showtext_auto, showtext_opts, str_replace, theme, and unit. The newsletter describing this visualization at a 30,000 ft view [can be found here]https://shop.riffomonas.org/posts/what-have-i-learned-by-recreating-other-people-s-visualizations). You can find the original article describing the ridgeline plot [here( https://ourworldindata.org/baby-boom-seven-charts) and the article that presented the drug overdose heatmap [here](https://www.nytimes.com/2025/01/30/upshot/black-men-overdose-deaths.html). The data he uses can be [found here](https://www.humanfertility.org/Country/Country?cntr=USA). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

```R
library(tidyverse)
library(showtext)
library(ggtext)

font_add_google("Libre Franklin", "franklin")
showtext_opts(dpi = 300)
showtext_auto()

# https://www.humanfertility.org/File/GetDocument/Files/USA/20250425/USAasfrRR.txt

read_table("USAasfrRR.txt", skip = 2) %>%
  rename_all(tolower) %>%
  mutate(age = str_replace(age, "[-+]", "") %>% as.numeric) %>%
#  filter(age > 13 & age < 46) %>% # does same thing as setting limit in coord_cartesian
  ggplot(aes(x = year, y = age, fill = asfr)) +
  geom_tile(color = "white") +
  geom_vline(xintercept = c(1945.5, 1964.5), linewidth = 0.3) +
  annotate(geom = "segment",
           x = 1945.5, xend = 1964.5, y = 44.5, linewidth = 0.3,
           arrow = arrow(length = unit(5, "pt"), angle = 20, 
                         ends = "both", type = "closed")) +
  scale_fill_viridis_c(option = "A", direction = -1) +
  coord_equal(
    expand = FALSE,
    ylim = c(13.5, 45.5),
    xlim = c(1932, NA)
    ) +
  scale_x_continuous(breaks = seq(1930, 2025, 10)) +
  scale_y_continuous(breaks = seq(10, 50, 5)) +
  labs(x = NULL, y = NULL,
       title = "The U.S. Baby Boom was between 1946 and 1964",
       caption = "**Data source:** Human Fertility Database (2024)",
       fill = "Age-specific fertility rate") +
  theme(
    text = element_text(family = "franklin", color = "black"),
    panel.background = element_blank(),
    panel.grid = element_blank(),
    legend.position = "bottom",
    legend.justification.bottom = "right",
    legend.key.height = unit(6, "pt"),
    legend.key.width = unit(30, "pt"),
    legend.title = element_text(size = 8, vjust = 1),
    legend.text = element_text(size = 7.5, margin = margin(t = 3)),
    legend.margin = margin(t = 0),
    legend.box.margin = margin(t = 0),
    axis.ticks = element_line(linewidth = 0.2),
    axis.ticks.length = unit(4, "pt"),
    plot.title.position = "plot",
    plot.title = element_text(face = "bold", size = 12),
    plot.caption.position = "plot",
    plot.caption = element_markdown(hjust = 0, color = "gray40", size = 8,
                                    margin = margin(t = 0))
  )

ggsave("fertility-heatmap.png", width = 7.44, height = 3.6)
```
