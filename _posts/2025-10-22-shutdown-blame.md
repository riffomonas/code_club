---
layout: post
title: "Using Visual Studio Code to recreate a New York Times slope plot with R's ggplot2 (CC375)"
blurb: "Pat's favorite type of figure"
author: "PD Schloss"
date: 2025-10-22 9:00
comments: false
youtube: wfX7viuy_3o
start_hash: 
end_hash: 
---

Pat recreates a diverging barplot from the Washington Post that shows which party Americans are blaming for the current government shutdown in this livestream. He recreated the figure using R, ggplot2, showtext, ggtext, and other tools from the tidyverse. The functions he used from these packages included aes, annotate, coord_cartesian, element_blank, element_geom, element_text, element_textbox_simple, facet_wrap, factor, font_add_google, geom_col, geom_text, ggplot, ggsave, if_else, labs, library, margin, mutate, paste0, rev, scale_fill_manual, showtext_auto, showtext_opts, theme, tribble, unique, and unit. You can find the code he developed in this episode below. The newsletter describing this visualization at a 30,000 ft view can be [found here](https://shop.riffomonas.org/posts/i-don-t-really-like-this-plot-but-i-am-curious-how-to-create-it-anyway). You can find the original article presenting the figure [for free here](https://archive.is/vSyct). Here's a [video critiquing](https://youtu.be/MFLfl683XIE) the original plot. If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

```R
library(tidyverse)
library(showtext)
library(ggtext)

font_add_google("Libre Franklin", "franklin")
showtext_opts(dpi = 300)
showtext_auto()

tribble(
  ~date, ~blamed, ~party, ~percentage,
  "Oct. 2025", "democrats", "democrats", 30,
  "Oct. 2025", "democrats", "none", 23,
  "Oct. 2025", "republicans", "republicans", 47,
  "Oct. 2025", "republicans", "none", 6,
  "Sept. 2023", "democrats", "democrats", 40,
  "Sept. 2023", "democrats", "none", 13,
  "Sept. 2023", "republicans", "republicans", 33,
  "Sept. 2023", "republicans", "none", 20,
  "Jan. 2019", "democrats", "democrats", 29,
  "Jan. 2019", "democrats", "none", 24,
  "Jan. 2019", "republicans", "republicans", 53,
  "Jan. 2019", "republicans", "none", 0,
  "Jan. 2018", "democrats", "democrats", 28,
  "Jan. 2018", "democrats", "none", 25,
  "Jan. 2018", "republicans", "republicans", 48,
  "Jan. 2018", "republicans", "none", 5,
  "Oct. 2013", "democrats", "democrats", 29,
  "Oct. 2013", "democrats", "none", 24,
  "Oct. 2013", "republicans", "republicans", 53,
  "Oct. 2013", "republicans", "none", 0,
  "Nov. 1995", "democrats", "democrats", 24,
  "Nov. 1995", "democrats", "none", 29,
  "Nov. 1995", "republicans", "republicans", 51,
  "Nov. 1995", "republicans", "none", 2
) %>%
  mutate(
    date = factor(date, levels = rev(unique(date))),
    pretty_label = if_else(party == "none", "", paste0(percentage, "%")),
    x_label = if_else(blamed == "democrats", 53 - percentage, percentage),
    hjust_label = if_else(blamed == "democrats", -0.2, 1.2)
  ) %>%
  ggplot(aes(x = percentage, y = date, fill = party)) +
  geom_col(show.legend = FALSE) +
  geom_text(
    aes(x = x_label, label = pretty_label, hjust = hjust_label),
    color = "white") +
  facet_wrap(~blamed) +
  scale_fill_manual(
    breaks = c("democrats", "republicans", "none"),
    values = c("#4A8F9B", "#E6823D", "#F5F5F5")
  ) +
  coord_cartesian(
    ylim = c(0.5, 7),
    expand = FALSE, clip = "off") +
  annotate(
    geom = "text", x = 53, y = 6.8, label = "BLAMED DEMOCRATS",
    hjust = 1, fontface = "bold", layout = 1, size = 7.5, size.unit = "pt"
  ) +
  annotate(
    geom = "text", x = 0, y = 6.8, label = "BLAMED REPUBLICANS",
    hjust = 0, fontface = "bold", layout = 2, size = 7.5, size.unit = "pt"
  ) +

  # geom_text(
  #   data = tibble(
  #     x = c(52, 1), y = 6.8, blamed = c("democrats", "republicans"),
  #     label = c("BLAMED DEMOCRATS", "BLAMED REPUBLICANS"), hjust = c(1, 0)),
  #   aes(x = x, y = y, label = label, hjust = hjust), inherit.aes = FALSE,
  #   fontface = "bold"
  # ) +
  labs(
    title = "How blame compares to past government shutdowns or near shutdowns",
    caption = "2023 Democratic option was \"Biden and the Democrats in Congress,\" 2018/2019 Republican option was \"Trump and Republicans in Congress,\" 2013 Democratic option was \"Obama\" and 1995 Democratic option was \"Clinton\'s administration\""
  ) +
  theme(
    text = element_text(family = "franklin"),
    geom = element_geom(family = "franklin", fontsize = 8),
    panel.grid = element_blank(),
    panel.background = element_blank(),
    axis.ticks = element_blank(),
    axis.title = element_blank(),
    axis.text.x = element_blank(),
    axis.text.y = element_text(color = "black", size = 7.5),
    strip.text = element_blank(),
    strip.background = element_blank(),
    panel.spacing.x = unit(6, "pt"),
    plot.margin = margin(t = 6, r = 13, b = 8, l = 12),
    plot.title.position = "plot",
    plot.title = element_text(face = "bold", size = 11, margin = margin(b = 9)),
    plot.caption.position = "plot",
    plot.caption = element_textbox_simple(size = 7.75, margin = margin(t = 10))
  )

ggsave("shutdown_blame.png", width = 1700, height = 808, unit = "px")

#ggsave("shutdown_blame.png", width = 6, height = 2.851765, unit = "in")
```