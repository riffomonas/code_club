---
layout: post
title: "Recreating a NYT heatmap of deaths by drug overdoses in R with ggplot2 (CC349)"
blurb: "Creating a heatmap"
author: "PD Schloss"
date: 2025-03-13 8:00
comments: false
youtube: t4bGrrmhDYs
start_hash: 
end_hash: 
---

Pat uses R to show how to recreate a heatmap published in the New York Times describing the increase in deaths among Black men in Chicago due to drug overdoses. To pull this off he uses the ggplot2, showtext, and ggtext packages. The functions he uses from these packages include aes, annotate, coord_cartesian, element_blank, element_line, element_text, font_add_google, geom_abline, geom_tile, geom_vline, ggplot, ggsave, labs, library, margin, element_markdown, read_tsv, scale_fill_gradientn, scale_x_continuous, scale_y_continuous, seq, showtext_auto, showtext_opts, theme, and unit. The newsletter describing this visualization at a 30,000 ft view can be [found here](https://shop.riffomonas.org/posts/annotating-heat-maps-to-highlight-effects-of-fentanyl-overdoses-using-the-ggplot2-r-package). The code he developed to extract the data from the original heatmap in the previous episode is available [here](https://www.riffomonas.org/code_club/2025-03-10-fentanyl-extract). You can find the original New York Times article [here](https://www.nytimes.com/2025/01/30/upshot/black-men-overdose-deaths.html). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Data

The data extracted in the previous episode and used here, can be [downloaded here](assets/data/chicago_drug_deaths.tsv). The code below has been modified to read the data directly from this file without needing to download it.


## Code

```R
library(tidyverse)
library(showtext)
library(ggtext)

font_add_google("Libre Franklin", "franklin", regular.wt = 500)
font_add_google("Libre Franklin", "franklin_thin", regular.wt = 300)
showtext_opts(dpi = 300)
showtext_auto()

read_tsv("https://riffomonas/code_club/assets/data/chicago_drug_deaths.tsv") %>%
  ggplot(aes(x = year, y = age, fill = n)) +
  geom_tile(color = "white") +
  geom_abline(intercept = c(-1951, -1970), slope = 1,
              linetype = "longdash", linewidth = 0.25) +
  
  # the following line wasn't in the video. it masks the diagonal lines that
  # extended into the margin between the y-axis ticks and the heatmap
  geom_vline(xintercept = 1988.45, color = "white") +
  annotate(geom = "text", family = "franklin_thin", hjust = 0,
           x = 2005.5,
           y = 31,
           label = "Men born from 1951 to 1970",
           size = 9, size.unit = "pt") +
  annotate(geom = "text", family = "franklin",
           x = 1987.25,
           y = 83.9,
           label = "AGE", 
           size = 6, size.unit = "pt") +
  scale_fill_gradientn(
    colours = c("#F4F3E8", "#FBCFB0", "#F7AC88", "#EB8772", "#B6525B",
                "#C34280", "#9A248E", "#6E1193", "#29088C", "#0D0887",
                "#0D0887", "#0C0887", "#0D0987"),
    values = c(0,    50, 100, 150, 200,
               250, 300, 350, 400, 450,
               500, 600, 735) / 735,
    labels = c(200, 400, 600),
    breaks = c(200, 400, 600)
  ) +
  scale_x_continuous(breaks = seq(1990, 2020, 5)) +
  scale_y_continuous(breaks = seq(10, 80, 10)) +
  coord_cartesian(expand = FALSE, clip = "off",
                  xlim = c(1988.4, NA)) +
  labs(
    title = "**Drug deaths** in Chicago among **Black men**",
    caption = "Source: Times/Banner analysis of N.C.H.S. mortality data",
    fill = "Deaths per 100k",
    x = NULL,
    y = NULL) +
  theme(
    text = element_text(family = "franklin"),
    legend.position = "bottom",
    legend.justification.bottom = "right",
    legend.margin = margin(r = 0),
    legend.ticks.length = unit(10, "pt"),
    legend.key.height = unit(5, "pt"),
    legend.key.width = unit(24, "pt"),
    legend.title = element_text(size = 7, vjust = 1),
    legend.text = element_text(size = 6, margin = margin(t = 3)),
    plot.title.position = "plot",
    plot.title = element_markdown(size = 10, family = "franklin_thin",
                                  margin = margin(b = 8)),
    plot.caption.position = "plot",
    plot.caption = element_text(hjust = 0, color = "gray30", size = 7),
    axis.text = element_text(size = 7, color = "black"),
    axis.ticks.length = unit(3.5, "pt"),
    axis.ticks = element_line(linewidth = 0.2),
    panel.grid = element_blank(),
    panel.background = element_blank(),
    plot.margin = margin(8, 8, 10, 10)
  )

ggsave("chicago_drug_deaths.png", width = 5, height = 6.2)
``` 