---
layout: post
title: "Creating a beeswarm plot in R with ggplot2 using several approaches (CC368)"
blurb: "Deceptively simple plot!"
author: "PD Schloss"
date: 2025-09-03 9:00
comments: false
youtube: jhrzFxYGhLo
start_hash: 
end_hash: 
---

A beeswarm plot is kind of like a jittered plot, kind of like a sina plot, and kind of like a dotplot. What are those and how do you create them in R? Check out this episode of Code Club. We'll be recreating a visual from the New York Times showing the expected change in congressional representation for the state of Texas before and after their controversial redistricting process. I'll recreate the original figure using R, ggplot2, rvest, ggtext, and other tools from the tidyverse. The functions I used from these packages include abs, aes, annotate, as.numeric, bind_rows, case_when, coord_cartesian, distinct, element_blank, element_markdown, element_text, facet_wrap, factor, font_add_google, geom_beeswarm, geom_dotplot, geom_point, geom_richtext, geom_vline, ggplot, ggsave, html_table, if_else, library, margin, mutate, pluck, position_jitter, read_csv, read_html, rename_all, scale_color_identity, scale_fill_identity, scale_x_continuous, select, showtext_auto, showtext_opts, str_remove, theme, tribble, and unit. You can find the code he developed in this episode below. The newsletter describing this visualization at a 30,000 ft view can be [found here](https://shop.riffomonas.org/posts/showing-the-effects-of-gerrymandering-with-ggplot2). You can find the original article presenting the figure [here](https://archive.is/9ZXHx). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

```R
library(tidyverse)
library(rvest)
library(ggbeeswarm)
library(ggtext)
library(showtext)

font_add_google("Libre Franklin", "franklin")
showtext_opts(dpi = 300)
showtext_auto()

previous <- read_html("https://en.wikipedia.org/wiki/2024_United_States_presidential_election_in_Texas") %>%
  html_table() %>%
  pluck(44) %>%
  rename_all(tolower) %>%
  mutate(district = str_remove(district, "\\D\\D") %>% as.numeric(),
         trump = str_remove(trump, "%") %>% as.numeric(),
         harris = str_remove(harris, "%") %>% as.numeric(),
         difference = trump - harris) %>%
  select(district, difference) %>%
  distinct()


new <- read_csv("https://datawrapper.dwcdn.net/LDF9R/3/data.csv") %>%
  select(district = District,
         difference = `margin-victory`)

text_annotation <- tribble(
  ~x, ~color, ~hjust, ~districting, ~label,
  -41, "#1275B7", 0, "Current districts", "**11 districts**<br>Harris +10 or greater",
  0, "black", 0.5, "Current districts", "**2 districts**<br>within a 10-pt margin",
  41, "#C93335", 1, "Current districts", "**25 districts**<br>Trump +10 or greater",
  -41, "#1275B7", 0, "Proposed districts", "**8 districts** (-3 districts)<br>Harris +10 or greater",
  0, "black", 0.5, "Proposed districts", "**0 districts** (-2 districts)<br>within a 10-pt margin",
  41, "#C93335", 1, "Proposed districts", "**30 districts** (+5 districts)<br>Trump +10 or greater"
) %>%
  mutate(districting = factor(districting,
                              levels = c("Current districts", "Proposed districts")))

bind_rows(previous = previous,
          new = new,
          .id = "districting") %>%
  mutate(
    districting = factor(districting, levels = c("previous", "new"),
                         labels = c("Current districts", "Proposed districts")),
    difference = case_when(difference < -40 ~ -40,
                           difference > 40 ~ 40,
                           TRUE ~ difference),
    fill = case_when(difference > 20 ~ "#C93335",
                      difference > 10 ~ "#DD7E7F",
                      difference > 0 ~ "#F4D7D6",
                      difference > -20 ~ "#72A9D6",
                      difference >= -40 ~ "#1275B7"),
    color = if_else(abs(difference) <= 10, "black", "white")
  ) %>%
  ggplot(aes(x = difference, y = districting, fill = fill, color = color)) +
  annotate(
    geom = "rect",
    xmin = -10, xmax = 10,
    ymin = 0.3, ymax = 1.7,
    fill = "gray90"
  ) +
  geom_vline(xintercept = 0, color = "gray", linewidth = 0.3) +
  # geom_point(position = position_jitter(seed = 19760620)) +
  # geom_dotplot(stackdir = "center") +
  geom_beeswarm(size = 3.5, cex = 8, shape = 21,
                method = "compactswarm",
                priority = "random") +
  geom_richtext(data = text_annotation,
                inherit.aes = FALSE, label.size = 0,
                label.padding = margin(0, 0, 0, 0),
                label.margin = margin(0, 0, 5, 0),
                lineheight = 1, size = 3.5, vjust = 0,
                aes(x = x, y = 1.7, label = label, 
                    color = color, hjust = hjust)) +
  
  facet_wrap(~districting, nrow = 2, scale = "free_y",
             axes = "all_x") +
  coord_cartesian(clip = "off", expand = FALSE,
                  xlim = c(-41, 41)) +
  scale_x_continuous(breaks = seq(-40, 40, 10),
                     labels = c("+40<br>or greater", "+30", "+20",
                                "+10<br>Harris", "0", "+10<br>Trump", "+20",
                                "+30", "+40<br>or greater")
                     ) +
  scale_fill_identity() +
  scale_color_identity() +
  theme(
    text = element_text(family = "franklin"),
    strip.text = element_text(face = "bold", hjust = 0, size = 10,
                              margin = margin(l = 0, b = 33)),
    strip.background = element_blank(),
    axis.title = element_blank(),
    axis.ticks = element_blank(),
    axis.text.y = element_blank(),
    axis.text.x = element_markdown(hjust = c(0, rep(0.5, 7), 1),
                                   size = 7.5),
    panel.grid.major.y = element_blank(),
    panel.grid.major.x = element_line(color = "gray", linewidth = 0.3,
                                      linetype = "dashed"),
    panel.grid.minor = element_blank(),
    panel.background = element_blank(),
    panel.spacing.y = unit(25, "pt")
  )

ggsave("tx_jerrymandering.png", width = 6, height = 4.91)
```