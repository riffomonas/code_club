---
layout: post
title: "Using readxl and dplyr to format messy data to show how much the poverty rate has changed (CC335)"
blurb: "National Poverty Awareness Month"
author: "PD Schloss"
date: 2025-01-23 8:00
comments: false
youtube: ryz6wD_gaZg
start_hash: 
end_hash: 
---

Pat creates a figure showing the change in the percent of people in the US who live in poverty for the past 45 years in honor of National Poverty Awareness Month. In the background of the plot he shows the poverty rate for each state in the same years. He pulls this off using tools from ggplot2, dplyr, readxl, and ggtext. The functions he uses from these packages include read_excel, filter, str_detect, mutate, str_replace, select, case_when, row_number, uncount, glue, drop_na, bind_cols, summarize, geom_point, geom_line, labs, scale_color_manual, scale_size_continuous, scale_y_continuous, scale_x_continuous, guides, theme_classic, theme. Pat's newsletter describing how he would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/tracking-trends-in-poverty-with-ggplot2). The xlsx file can be downloaded [from here](https://www2.census.gov/programs-surveys/cps/tables/time-series/historical-poverty-people/hstpov19.xlsx). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)
library(readxl)
library(glue)
library(ggtext)

raw_data <- read_excel("hstpov19.xlsx", col_names = FALSE)

year_tibble <- raw_data %>%
  filter(str_detect(`...1`, "^\\d")) %>%
  mutate(`...1` = str_replace(`...1`, " \\(\\d+\\)", "")) %>%
  select(year = `...1`) %>%
  mutate(year = as.numeric(year),
         method = case_when(row_number() <= 7 ~ "C",
                            row_number() >= 13 ~ "A",
                            year >= 2013 & year <= 2017 ~ "B"),
         n = 51) %>%
  uncount(weights = n)

clean_data <-raw_data %>%
  drop_na() %>%
  filter(str_detect(`...2`, "^\\d")) %>%
  select(state = `...1`,
         total = `...2`,
         number = `...3`,
         percent = `...5`) %>%
  mutate(total = as.numeric(total),
         number = as.numeric(number),
         percent = as.numeric(percent)) %>%
  bind_cols(year_tibble)

state_data <- clean_data %>%
  summarize(total = mean(total),
            percent = mean(percent),
            .by = c(state, year))

national_data <- clean_data %>%
  summarize(percent = mean(percent),
            .by = c(year, method))

min_max <- national_data %>%
  summarize(min = min(percent) %>% round(digits = 1) %>% format(nsmall = 1),
            max = max(percent) %>% round(digits = 1) %>% format(nsmall = 1)
            )

state_data %>%
  ggplot(aes(x = year, y = percent, size = total)) +
  geom_point(color = "gray70", alpha = 0.25,
             show.legend = FALSE) +
  geom_line(data = national_data,
            mapping = aes(x = year, y = percent, color = method),
            linewidth = 2, lineend = "round",
            inherit.aes = FALSE, show.legend = FALSE) +
  labs(x = "Year",
       y = "Poverty rate (%)",
       title = glue("The average national poverty rate has varied between {min_max$min} and {min_max$max}% since 1980"),
       subtitle = "Poverty rate at the state level is indicated by individual points, which are sized to their overall population size",
       caption = "Data collected and reported by the US Census Bureau in 2024<br>Table 10 from https:\\/\\/www.census.gov/data/tables/time-series/demo/income-poverty/historical-poverty-people.html") +
  scale_color_manual(breaks = c("A", "B", "C"),
                     values = c("darkgreen", "green4", "darkgreen")) +
  scale_size_continuous(range = c(0.5, 4)) +
  scale_y_continuous(breaks = seq(0, 30, 5),
                     limits = c(0, 31),
                     expand = c(0, 0)) +
  scale_x_continuous(breaks = seq(1980, 2020, 5),
                     minor_breaks = 1980:2023,
                     limits = c(1979, 2024),
                     expand = c(0, 0)) +
  guides(
    x = guide_axis(minor.ticks = TRUE)
  ) +
  theme_classic() +
  theme(
    plot.title.position = "plot",
    plot.title = element_textbox_simple(size = 20, color = "darkgreen",
                                        face = "bold", lineheight = 1),
    plot.subtitle = element_textbox_simple(size = 12, color = "gray50",
                                           margin = margin(t = 10, b = 10)),
    plot.caption.position = "plot",
    plot.caption = element_textbox_simple(hjust = 0, color = "gray70",
                                          margin = margin(t = 10)),
    axis.title = element_text(color = "gray70"),
    axis.text.y =  element_text(color = "gray70"),
    axis.text.x =  element_text(color = "gray70", margin = margin(t = 7)),
    axis.line = element_line(color = "gray70"),
    axis.ticks.y = element_blank(),
    axis.ticks.x = element_line(color = "gray70", linewidth = 0.25),
    axis.ticks.length.x = unit(-5, "pt"),
    axis.minor.ticks.length.x = unit(-3, "pt"),
    panel.grid.major.y = element_line(color = "gray90", linewidth = 0.25)
  )

ggsave("poverty_by_state.png", height = 5, width = 7)
```
