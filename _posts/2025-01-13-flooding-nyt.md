---
layout: post
title: "Recreating a New York Times bar chart using the ggplot2 R package (CC332)"
blurb: "Highlighting data with color"
author: "PD Schloss"
date: 2025-01-13 8:00
comments: false
youtube: 8_2frsX5MwM
start_hash: 
end_hash: 
---

Pat recreates a visualization from the New York Times showing the increase in flooding declarations as a bar chart using the ggplot2, ggtext, and showtext R packages. This barplot is distinctive because it highlights one column with a different color. To recreate off this visualization, Pat uses geom_col, geom_richtext, scale_fill_manual, coord_cartesian, labs, theme_classic, and theme. The original NYT article can be [found here](https://www.nytimes.com/2024/10/22/briefing/americas-flooding-problem.html). Pat's newsletter describing how he would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/flooding-you-opportunity-to-practice-your-data-viz-skills-in-r). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)
library(ggtext)
library(showtext)

font_add_google("Domine", "domine")
font_add_google("Libre Franklin", "franklin")

showtext_auto()
showtext_opts(dpi = 300)

flooding <- tibble(
  year = 2000:2024,
  declarations = c(2, 5, 4, 0, 1, 2, 2, 1, 3, 3, 9, 25,
                   3, 17, 10, 9, 18, 12, 17, 53, 21, 10,
                   14, 42, 66),
  is_2024 = year == 2024
)

flooding %>%
  ggplot(aes(x = year, y = declarations, fill = is_2024)) +
  annotate(geom = "segment",
           x = 2022.5, xend = 2024,
           y = 63.5, yend = 63.5) +
  annotate(geom = "text",
           x = 1998.5,
           y = c(20, 40, 60) + 1,
           label = c("20", "40", "60 declared flood disasters"),
           hjust = 0,
           size = 11, size.unit = "pt",
           family = "franklin",
           color = "gray40") +
  geom_col(show.legend = FALSE) +
  geom_richtext(label = "<span style='color:#539CBA'>**66 declarations**</span><br>this year so far",
                x = 2022.5, y = 62,
                hjust = 1,
                fill = NA,
                color = "black",
                label.colour = NA,
                family = "franklin",
                show.legend = FALSE) +
  scale_fill_manual(breaks = c(FALSE, TRUE),
                    values = c("#BAD1D6", "#539CBA")) +
  coord_cartesian(xlim = c(1998.5, 2024.5),
                  expand = FALSE) +
  labs(title = "A surge in U.S. flood disasters",
       caption = "Note: The 2024 total reflects declarations as of Oct. 22, 2024. • Source: Federal Emergency Management Agency • By The New York Times",
       x = NULL,
       y = NULL) +
  theme_classic() +
  theme(
    text = element_text(family = "franklin"),
    plot.title = element_text(face = "bold", family = "domine", size = 16,
                              margin = margin(b = 15)),
    plot.caption = element_textbox_simple(margin = margin(t = 25), size = 9,
                                          color = "gray40"),
    plot.caption.position = "plot",
    plot.title.position = "plot",
    axis.line.x = element_line(color = "lightgray", linewidth = 0.25),
    axis.ticks.x = element_line(color = "lightgray", linewidth = 0.25),
    axis.ticks.length.x = unit(6, "pt"),
    axis.line.y = element_blank(),
    axis.ticks.y = element_blank(),
    axis.text.y = element_blank(),
    axis.text.x = element_text(color = "gray40", size = 10),
    panel.grid.major.y = element_line(color = "lightgray", linewidth = 0.25)
  )

ggsave("flooding.png", width = 6.38, height = 5.80)
```
