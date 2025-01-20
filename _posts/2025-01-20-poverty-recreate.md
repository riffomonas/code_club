---
layout: post
title: "Using ggplot2 to see how much has the rate of poverty changed in the US over the past 65 years (CC334)"
blurb: "National Poverty Awareness Month"
author: "PD Schloss"
date: 2025-01-20 8:00
comments: false
youtube: biPe61VD8oY
start_hash: 
end_hash: 
---

Pat recreates a figure showing the change in the number and percent of people in the US who live in poverty for the past 65 years in honor of National Poverty Awareness Month. He pulls this off using tools from ggplot2, dplyr, readxl, jsonlite, ggtext, and showtext. The functions he uses from these packages include font_add_google, showtext_auto, showtext_opts, select, filter, mutate, pivot_longer, inner_join, str_replace, case_when, row_number, paste, year, yday, read_excel, geom_rect, geom_hline, geom_line, geom_text, facet_wrap, labeller, labs, scale_color_manual, scale_fill_manual, scale_y_continuous, scale_x_continuous, coord_cartesian, guides, guide_axis, theme_classic, and theme. Pat's newsletter describing how he would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/tracking-trends-in-poverty-with-ggplot2). The recession data can be [found here](https://data.nber.org) and the poverty data can be [found here](https://www.census.gov/data/tables/time-series/demo/income-poverty/historical-poverty-people.html). The xlsx file can be [downloaded from here](https://www2.census.gov/programs-surveys/cps/tables/time-series/historical-poverty-people/hstpov2.xlsx). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)
library(jsonlite)
library(readxl)
library(ggtext)
library(showtext)

font_add_google("Libre Franklin", "franklin")
showtext_auto()
showtext_opts(dpi = 300)

facet_labels <- c(
  number = "Number in poverty (millions)",
  percent = "Poverty rate (%)"
)

grid_lines <- tibble(
  name = c(rep("number", 7), rep("percent", 6)),
  intercept = c(seq(20, 50, 5), seq(0, 25, 5))
)


cycle_limits <- tibble(name = c("number", "percent"),
                        ymin = c(20, 0),
                        ymax = c(50, 25))

# Thanks Rob Hanssen!
cycles <- jsonlite::read_json("https://data.nber.org/data/cycles/business_cycle_dates.json", simplifyVector = TRUE) %>%
    mutate(across(everything(), ymd)) %>%
  select(beginning = peak, end = trough) %>%
  filter(year(beginning) >= 1959) %>%
    mutate(beginning = year(beginning) + yday(beginning) / 365,
           end = year(end) + yday(end) / 365,
           number = "x",
           percent = "y") %>%
    pivot_longer(cols = c(number, percent)) %>%
  inner_join(., cycle_limits, by = "name")

# All tables available at:
# https://www.census.gov/data/tables/time-series/demo/income-poverty/historical-poverty-people.html
# Saved this to Desktop:https://www2.census.gov/programs-surveys/cps/tables/time-series/historical-poverty-people/hstpov2.xlsx
poverty_data <- read_excel("hstpov2.xlsx",
                       range = "A9:D75",
                       col_names = c("year", "total", "number", "percent")) %>%
  mutate(year = str_replace(year, " \\(.*\\)", "") %>% as.numeric(),
         method = case_when(row_number() >= 8 & row_number() <= 12 ~ "B",
                            year >= 2017 ~ "C",
                            year <= 2013 ~ "A"),
         number = number / 1000) %>%
  select(-total) %>%
  pivot_longer(cols = c(number, percent)) %>%
  mutate(method_name = paste(method, name, sep = "_"))


text_annotation <- poverty_data %>%
  filter(year == 2023) %>%
  mutate(year = year + 1,
         label = paste(value, c("million", "percent")))

poverty_data %>%
  ggplot(aes(x = year, y = value, color = method_name, group = method_name)) +
  geom_rect(data = cycles,
            mapping = aes(xmin = beginning, xmax = end,
                          ymin = ymin, ymax = ymax, fill = "fill_color"),
            inherit.aes = FALSE) +
  geom_hline(data = grid_lines, aes(yintercept = intercept),
             linewidth = 0.25, color = "gray")+
  geom_line(show.legend = FALSE, linewidth = 1, lineend = "round") +
  geom_text(data = text_annotation,
            mapping = aes(x = year, y = value, label = label),
            hjust = 0, size = 8, size.unit = "pt", family = "franklin",
            inherit.aes = FALSE) +
  facet_wrap(~name, nrow = 2, scale = "free_y",
             labeller = labeller(name = facet_labels)) +
  labs(x = NULL, y = NULL,
       title = "Figure 1.",
       subtitle = "Number in Poverty and Poverty Rate Using the Official Poverty Measure: 1959 to 2023",
       caption = "Note: Population as of March of the following year. The data for 2017 and beyond reflect the implementation of an updated processing system. The data for 2013 and beyond reflect the implementation of the redesigned income questions. Refer to Table A-3 for historical footnotes. The data points are placed at the midpoints of the respective years. Information on recessions is available in Appendix C.<br>
Information on confidentiality protection, sampling error, nonsampling error, and definitions is available at census.gov.<br>
Source: U.S. Census Bureau, Current Population Survey, 1960 to 2024 Annual Social and Economic Supplements (CPS ASEC).") +
  scale_color_manual(
    breaks = c("A_number", "B_number", "C_number",
               "A_percent", "B_percent", "C_percent"),
    values = c("purple", "violet", "purple",
               "forestgreen", "limegreen", "forestgreen")
  ) +
  scale_fill_manual(name = NULL,
                    breaks = "fill_color",
                    label = "Recession",
                    values = "lightblue") +
  scale_y_continuous(breaks = seq(0, 50, 5),
                     expand = c(0, 0)) +
  scale_x_continuous(breaks = seq(1960, 2020, 5),
                     minor_breaks = 1959:2023,
                     expand = c(0, 0)) +
  coord_cartesian(clip = "off",
                  xlim = c(1958.5, 2023.5)) +
  guides(
    x = guide_axis(minor.ticks = TRUE)
  ) +
  theme_classic() +
  theme(
    plot.margin = margin(3, 60, 3, 3),
    text = element_text(family = "franklin"),
    plot.title.position = "plot",
    plot.caption.position = "plot",
    plot.caption = element_textbox_simple(hjust = 0, size = 6,
                                          margin = margin(t = 15, r = -60)),
    plot.title = element_text(size = 7),
    plot.subtitle = element_text(size = 8, face = "bold", color = "dodgerblue"),
    strip.background = element_blank(),
    strip.text = element_text(size = 7, hjust = 0, margin = margin(l = 0, b = 3)),
    axis.text = element_text(size = 7),
    axis.line = element_line(linewidth = 0.25),
    axis.line.y.right = element_line(linewidth = 0.25, color = "gray"),
    axis.ticks.x = element_line(linewidth = 0.25),
    axis.ticks.y = element_blank(),
    axis.minor.ticks.x.bottom = element_line(),
    axis.minor.ticks.length.x.bottom = unit(-2, "pt"),
    axis.ticks.length.x.bottom = unit(-4, "pt"),
    panel.border = element_rect(color = "gray", linewidth = 0.25, fill = NA),
    legend.text = element_text(size = 7),
    legend.key.height = unit(9, "pt"),
    legend.position = "inside", 
    legend.position.inside = c(0.92, 1.03),
    legend.background = element_blank()
  )

ggsave("poverty_trends.png", width = 6, heigh = 4.34)
```
