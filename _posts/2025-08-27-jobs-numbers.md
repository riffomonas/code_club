---
layout: post
title: "Using ggplot2 to display the revised change in US Jobs Numbers for May and June (CC367)"
blurb: "Jobs data gets political"
author: "PD Schloss"
date: 2025-08-27 9:00
comments: false
youtube: whVAKiZx6rw
start_hash: 
end_hash: 
---

This summer the US Bureau of Labor Statistics revised their projections for the number of jobs created in in May and June. The New York Times had an interesting visualization of the data. I decided to recreate the original using R, readxl, ggplot2, and other tools from the tidyverse. The functions I used from these packages include aes, annotate, arrange, as.character, bind_rows, c, element_blank, element_line, element_text, factor, filter, font_add_google, function, geom_col, ggplot, ggsave, if_else, labs, lag, library, margin, month, mutate, pivot_longer, position_identity, read_excel, rename, scale_color_manual, scale_fill_manual, scale_x_discrete, scale_y_continuous, select, showtext_auto, showtext_opts, theme, tibble, unit, and ymd. The newsletter describing this visualization at a 30,000 ft view can be [found here](https://shop.riffomonas.org/posts/plotting-the-us-job-creation-numbers-and-revisions-with-ggplot2). You can find the original article presenting the [figure here](https://archive.is/R2UEJ). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



```R
library(tidyverse)
library(readxl)
library(scales)
library(showtext)

font_add_google("Libre Franklin", "franklin", bold.wt = 500)
showtext_opts(dpi = 300)
showtext_auto()

parse_month <- function(x) {
  month(x, abbr = TRUE, label = TRUE)
}

read_excel("SeriesReport-20250827091254_d212f6.xlsx",
           range = "A13:M24") %>%
  pivot_longer(-Year, values_to = "number_of_jobs", names_to = "month") %>%
  rename(year = Year) %>%
  mutate(new_jobs = number_of_jobs - lag(number_of_jobs),
         date = ymd(paste(year, month, 1, sep = "-")),
         type = if_else(date != "2025-07-01", "real", "july")) %>% #lubridate
  filter(date >= "2024-07-01" & date <= "2025-07-01") %>%
  bind_rows(
    tibble(year = c(2025, 2025),
           month = c("May", "June"),
           date = ymd("2025-05-01", "2025-06-01"),
           new_jobs = c(144, 147),
           type = "preliminary")
  ) %>%
  select(date, new_jobs, type) %>%
  mutate(type = factor(type, levels = c("preliminary", "real", "july")),
         date = as.character(date)) %>%
  arrange(date, type) %>%
  ggplot(aes(x = date, y = new_jobs, fill = type, color = type)) +
  geom_col(position = position_identity(), linewidth = 0.3, alpha = 0.9) +
  annotate(
    geom = "text", hjust = -0.05, family = "franklin",
    x = -0.4, y = c(108, 208, 308),
    label = c("+100,000", "+200,000", "+300,000"),
    size = 8.5, size.unit = "pt", color = "gray40") +
  annotate(
    geom = "label",
    x = c(11.5, 7.75, 13.5),
    y = c(95, 200, 270),
    label = c("REVISED\nDOWN", "Growth in May and\nJune was lower than\ninitially estimated.", "+73,000 jobs\nin July"),
    hjust = c(0.5, 0, 1),
    color = c("gray40", "black", "black"),
    fontface = c("bold.italic", "bold", "bold"),
    vjust = 0.5, alpha = 0.8, lineheight = 1,
    label.size = 0, label.padding = unit(0, "pt"), family = "franklin",
    size = 9, size.unit = "pt"
  ) +
  annotate(
    geom = "segment",
    x = 13, y = 73, yend = 240, linewidth = 0.4
  ) +
  annotate(
    geom = "curve",
    x = 10.9, xend = 11.5, y = 180, yend = 155,
    linewidth = 0.4, curvature = -0.5
  ) +
  scale_fill_manual(
    breaks = c("preliminary", "real", "july"),
    values = c("#FFFFFF", "gray70", "#FE8918")
  ) +
  scale_color_manual(
    breaks = c("preliminary", "real", "july"),
    values = c("#BDBDBD", "#BDBDBD", "#FE8918")
  ) +
  scale_x_discrete(
    breaks = c("2024-07-01", "2024-09-01", "2024-11-01",
               "2025-01-01", "2025-03-01", "2025-05-01", "2025-07-01"),
  #  labels = parse_month
    labels = c("July '24", "Sept.", "Nov.",
               "Jan. '25", "March", "May", "July")
  ) +
  scale_y_continuous(expand = c(0, 0)) +
  labs(
    title = "Monthly change in jobs",
    caption = "Source: Bureau of Labor Statistics • Note: Data is seasonally adjusted. • By Christine Zhang/Pat Schloss"
  ) +
  theme(
    text = element_text(family = "franklin"),
    panel.background = element_blank(),
    panel.grid.minor = element_blank(),
    panel.grid.major.x = element_blank(),
    panel.grid.major.y = element_line(color = "gray", linewidth = 0.4,
                                      linetype = "12"),
    axis.line.x = element_line(linewidth = 0.3),
    axis.ticks = element_blank(),
    axis.title = element_blank(),
    axis.text.x = element_text(color = "black"),
    axis.text.y = element_blank(),
    legend.position = "none",
    plot.title = element_text(color = "gray40", size = 10, face = "bold",
                              margin = margin(l = -15, b = 10)),
    plot.caption = element_text(color = "gray40", size = 8, hjust = 0,
                                face = "bold",
                                margin = margin(t = 10, l = -15)),
    plot.margin = margin(t = 8, r = 15, b = 8, l = 20)
  )

ggsave("jobs_numbers.png", width = 6, height = 3.6)
```