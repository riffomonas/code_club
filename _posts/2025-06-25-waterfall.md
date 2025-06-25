---
layout: post
title: "Visualizing contributions to the US GDP as a Washington Post waterfall plot using ggplot2 (CC361)"
blurb: "Four geom_segment calls!"
author: "PD Schloss"
date: 2025-06-25 8:00
comments: false
youtube: IzOLk-NihXY
start_hash: 
end_hash: 
---

Pat recreates a waterfall chart from the Washington Post to show the factors that accounted for the drop in the Q1 2025 GDP in the US using ggplot2. The plot was generated using functions from the tidyverse including the dplyr, ggplot2, showtext, ggtext, and glue packages. The functions he uses from these packages include abs, aes, annotate, arrow, coord_cartesian, cumsum, element_blank, element_line, element_text, factor, filter, font_add_google, geom_hline, geom_richtext, geom_segment, ggplot, ggsave, glue, if_else, labs, lag, lead, library, margin, mutate, pull, round, scale_color_identity, scale_y_continuous, showtext_auto, showtext_opts, sum, theme, tribble, and unit. You can find the code he developed in this episode [here](https://www.riffomonas.org/code_club/2025-06-25-waterfall). The newsletter describing this visualization at a 30,000 ft view can be [found here](https://shop.riffomonas.org/posts/don-t-go-chasing-waterfall-charts). You can find the original article describing the ridgeline plot [here](https://archive.is/gkPZX). The data he uses can be found [here](https://www.bea.gov/news/2025/gross-domestic-product-1st-quarter-2025-advance-estimate). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

```R
library(tidyverse)
library(showtext)
library(ggtext)
library(glue)

font_add_google("Libre Franklin", "franklin")
showtext_opts(dpi = 300)
showtext_auto()

# x data: https://www.bea.gov/sites/default/files/2025-04/gdp1q25-adv.pdf
# x "floating barplot" - fat, colored geom_segment lines
# x arrows within rectangles - arrowed, black geom_segment lines
# x clear background out / removing x-axis styling / y-axis styling
# x title, caption, attribution
# x "caps" on floating barplot
# x annotations with bolded color and black font

x_adj <- 0.467
arrow_adj <- 0.015

data <- tribble(
  ~category, ~change, ~color,
  "personal_consumption", 1.21, "#C8E8EF",
  "private_investment", 3.60, "#C8E8EF",
  "gov_spending", -0.25, "#E5B3D3",
  "exports", 0.19, "#FBC296",
  "imports", -5.03, "#FBC296"
) %>%
  mutate(category = factor(category, levels = category),
         yend = cumsum(change),
         y = lag(yend, default = 0),
         x = 1:5,
         xend = lead(x, default = 5),
         yend_arrow = if_else(change > 0,
                              yend - arrow_adj, yend + arrow_adj))

investment_incr <- filter(data, category == "private_investment") %>%
  pull(change) %>%
  round(1)

import_drop <- filter(data, category == "imports") %>%
  pull(change) %>%
  round(1) %>% abs()

overall_drop <- round(sum(data$change), 1)

labels <- tribble(
  ~x, ~y, ~hjust, ~label,
  0.52, 0.7, 1, "<span style = 'color:#197487;'>**Personal<br>consumption**</span>",
  1.52, 3.6, 1, glue("<span style = 'color:#197487;'>**Private<br>investment**</span><br>in equipment and<br>inventories pushed<br>GDP up {investment_incr} ppt"),
  2.5, 4.5, 0, "<span style = 'color:#AA3192;'>**Government<br>spending**</span><br>shrank slightly",
  3.5, 4.99, 0, "<span style = 'color:#CF6B2D;'>**Exports**</span>",
  5.5, 2.8, 0, glue("<span style = 'color:#CF6B2D;'>**Imports**</span> surged as<br>businesses tried to get<br>ahead of tariffs, pushing<br>GDP down {import_drop} ppt</span>"),
  4.5, -0.35, 0, glue("<span style = 'color:#000000;'>**{overall_drop}% OVERALL<br>CHANGE**</span>")
)

data %>%
  ggplot(aes(x = x, y = y, yend = yend, color = color)) +
  geom_hline(yintercept = 0, linewidth = 0.5, color = "black") +
  geom_segment(linewidth = 18) +
  geom_segment(aes(yend = yend_arrow),
               linewidth = 0.6, color = "black",
               arrow = arrow(angle = 45, length = unit(7, "pt"))) +
  geom_segment(aes(x = x - x_adj, xend = xend + x_adj, y = yend),
               color = "black") +
  geom_segment(x = 1 - x_adj, xend = 1 + x_adj, y = 0, yend = 0,
               color = "black", linewidth = 0.5) +
  geom_richtext(data = labels, inherit.aes = FALSE,
                aes(x = x, y = y, label = label, hjust = hjust),
                label.padding = unit(2, "pt"),
                label.size = unit(0, "pt"), family = "franklin",
                size = 3.5, vjust = 1) +
  annotate(geom = "label",
           x = c(-1.42, 8.2),
           y = c(5, -0.92),
           label = c("percentage points", "PAT SCHLOSS/RIFFOMONAS"),
           vjust = 0.5, hjust = c(0, 1), family = "franklin",
           label.size = unit(0, "pt"),
           # label.padding = unit(0, "pt"),
           size = 8.25, size.unit = "pt") +
  labs(title = "Contributions to quarterly GDP change in Q1 2025",
       caption = "Source: Bureau of Economic Analysis") +
  coord_cartesian(expand = FALSE, clip = "off",
                  xlim = c(-1.3, 8.1),
                  ylim = c(-0.75, 5)) +
  scale_y_continuous(
    breaks = 0:5,
    labels = c("0", "+1", "+2", "+3", "+4", "+5")) +
  scale_color_identity() +
  theme(
    text = element_text(family = "franklin"),
    panel.grid.major.x = element_blank(),
    panel.grid.major.y = element_line(color = "gray70", linewidth = 0.2),
    panel.grid.minor = element_blank(),
    panel.background = element_blank(),
    axis.ticks = element_blank(),
    axis.text.x = element_blank(),
    axis.text.y = element_text(hjust = 1, size = 8.25, color = "black"),
    axis.title = element_blank(),
    plot.title.position = "plot",
    plot.title = element_text(size = 9, margin = margin(b = 12)),
    plot.caption.position = "plot",
    plot.caption = element_text(hjust = 0, size = 8),
    plot.margin = margin(t = 9, r = 16, b = 5, l = 13)
  )

ggsave("waterfall-plot.png", width = 6, height = 5.80)
```
