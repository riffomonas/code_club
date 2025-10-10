---
layout: post
title: "Creating stacked area plots with ggplot2 and patchwork (CC372)"
blurb: "Trying Positron and ggplot2 yet again"
author: "PD Schloss"
date: 2025-10-10 9:00
comments: false
youtube: 2NCjG5ruWxo
start_hash: 
end_hash: 
---

Pat recreates a figure from the Financial Times that has two area plots. He decided to use the patchwork package to recreate this figure with ggplot2. He recreated the figure using R, ggplot2, showtext, ggtext, patchwork, readxl, and other tools from the tidyverse. The functions he used from these packages included aes, coord_cartesian, cumsum, element_blank, element_line, element_rect, element_text, element_textbox_simple, factor, filter, font_add_google, geom_area, geom_richtext, ggplot, ggsave, guide_legend, guides, labs, library, mutate, pivot_longer, plot_annotation, plot_layout, read_excel, replace_na, rev, scale_fill_manual, scale_x_continuous, scale_y_continuous, select, seq, showtext_auto, showtext_opts, sum, tibble, and unit. The newsletter describing this visualization at a 30,000 ft view can be [found here](https://shop.riffomonas.org/posts/visualizing-contributions-to-global-co2-production-along-side-the-financial-times). You can find the original article presenting the figure [here](https://ep.ft.com/permalink/emails/eyJlbWFpbCI6ImRmNDIwMThhNmVlMTVmZTQ5OTYwZjk3ZjkwNjI2ZTRiZmQ5ZCIsICJ0cmFuc2FjdGlvbklkIjoiZTNmMWJlNzAtMzlhMS00YTljLTk2ZWUtOTQ1NWExMWY2YTM4IiwgImJhdGNoSWQiOiI4MWRiYzU0Mi0yNDNjLTQ5ODYtYjM0Yy1iNTE5ZWViZDNhMjYifQ==). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!


```R
library(tidyverse)
library(readxl)
library(ggtext)
library(showtext)
library(patchwork)

font_add_google("Libre Franklin", "franklin", regular.wt = 500)
showtext_opts(dpi = 300)
showtext_auto()

regions <- c("China", "US", "EU27", "Russia", "UK", "Other")

co2_data <- read_excel(
    "National_Fossil_Carbon_Emissions_2023v1.0.xlsx",
    sheet = "Territorial Emissions", skip = 11) %>%
  select(
    year = "...1", "China", US = "USA",
    "EU27", UK = "United Kingdom", "Russia", "World") %>%
  mutate(
    China = replace_na(China, 0),
    Other = World - (China + US + EU27 + UK + Russia)
  ) %>%
  select(-World) %>%
  pivot_longer(-year, names_to = "region", values_to = "c") %>%
  mutate(
    co2 = c * 3.664 / 1000,
    region = factor(
      region,
      levels = rev(regions))
  ) %>%
  select(-c)

left_plot <- co2_data %>%
  filter(year >= 1950) %>%
  ggplot(aes(x = year, y = co2, fill =region)) +
  geom_area(show.legend = TRUE) +
  scale_fill_manual(
    breaks = regions,
    values = c(
      "#BC3C00", "#F05828", "#118DA2",
      "#4CBABB", "#8CE6E1", "#DACDC3")
  ) +
  scale_y_continuous(
    breaks = seq(0, 40, 10), expand = c(0, 0)
  ) +
  scale_x_continuous(
    breaks = seq(1960, 2030, 20), expand = c(0.02, 0)
  ) +
  coord_cartesian(xlim = c(1950, NA), ylim = c(0, 40 * 1.1), clip = "off") +
  labs(title = NULL, x = NULL, y = NULL, fill = NULL) +
  guides(fill = guide_legend(nrow = 1)) +
  geom_richtext(
    data = tibble(year = 1942, co2 = 44, region = "US"),
    label = "Annual (Gt CO<sub>2</sub>)",
    hjust = 0, vjust = 1, label.padding = unit(c(0, 0, 0, 0), "pt"),
    label.color = NA, fill = NA,
    color = "#666666", size = 4, family = "franklin"
)

right_plot <- co2_data %>%
  mutate(cum_co2 = cumsum(co2), .by = region) %>%
  mutate(cum_relative_co2 = 100 * cum_co2 / sum(cum_co2), .by = year) %>%
  ggplot(aes(x = year, y = cum_relative_co2, fill = region)) +
  geom_area(show.legend = TRUE) +
    scale_fill_manual(
    breaks = regions,
    values = c(
      "#BC3C00", "#F05828", "#118DA2",
      "#4CBABB", "#8CE6E1", "#DACDC3")
  ) +
  scale_y_continuous(
    breaks = c(0, 50, 100), expand = c(0, 0)
  ) +
  scale_x_continuous(
    breaks = seq(1850, 2030, 50), expand = c(0.02, 0)
  ) +
  coord_cartesian(ylim = c(0, 100 * 1.10), xlim = c(1850, NA), clip = "off") +
  labs(title = NULL, x = NULL, y = NULL, fill = NULL) +
  guides(fill = guide_legend(nrow = 1)) +
  geom_richtext(
    data = tibble(year = 1825, cum_relative_co2 = 100 * 44/40, region = "US"),
    label = "Cumulative Historical Share (%)",
    hjust = 0, vjust = 1, label.padding = unit(c(0, 0, 0, 0), "pt"),
    label.color = NA, fill = NA,
    color = "#666666", size = 4, family = "franklin"
)


patchwork <- (left_plot | right_plot) &
  theme(
    text = element_text(family = "franklin"),
    plot.background = element_rect(fill = "#FFFAF4", color = "#FFFAF4"),
    legend.background = element_blank(),
    legend.direction = "horizontal",
    legend.position = "top",
    legend.justification.top = "left",
    legend.text = element_text(
      size = 11, color = "gray40", margin = margin(l = 4, r = 6)),
    legend.margin = margin(0,0,10,-15),
    legend.key.height = unit(11, "pt"),
    legend.key.width = unit(20, "pt"),

    axis.text = element_text(color = "#666666", size = 11),
    axis.text.x = element_text(margin = margin(t = 7)),
    panel.background = element_blank(),
    panel.grid.minor = element_blank(),
    panel.grid.major.x = element_blank(),
    panel.grid.major.y = element_line(color = "#DACDC3"),
    axis.ticks.y = element_blank(),
    axis.ticks.x = element_line(color = "#DACDC3"),
    axis.ticks.length.x = unit(5, "pt")
  )

patchwork   +
  plot_annotation(
  title = "China accounts for almost a third of current global\nemissions, with a cumulative share of 15%",
  subtitle = "Fossil carbon dioxide emissions",
  caption = "Source: Global Carbon Budget <span style='font-family:wqy-microhei;'>\U25CF</span> Includes carbon dioxide emissions from fossil\nfuels and fossil carbonates<br>&copy; *FT*",
  theme = theme(
    text = element_text(family = "franklin"),
    plot.title = element_text(
      face = "bold", size = 16, margin = margin(t = 15), lineheight = 1.2),
    plot.subtitle = element_text(
      color = "gray40", size = 15, margin = margin(t = 10, b = 8)),
    plot.caption = element_textbox_simple(
      hjust = 0, color = "#666666", size = 11, lineheight = 1.5,
      margin = margin(t = 15), family = "franklin"),
    plot.background = element_rect(fill = "#FFFAF4")
    )
) +
plot_layout(guides = "collect")

ggsave("global_co2_budget.png", width = 6, height = 6)
```