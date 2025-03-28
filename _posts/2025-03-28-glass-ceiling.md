---
layout: post
title: "Visualizing the The Economist's Glass Ceiling Index in R with ggplot2 and ggborderline (CC353)"
blurb: "Fun with geom_borderline and geom_text"
author: "PD Schloss"
date: 2025-03-28 8:00
comments: false
youtube: cWX6JBwFEq4
start_hash: 
end_hash: 
---

Pat uses R to recreate the variation in The Economist's Glass Ceiling Index between 2016 and 2024 with help from the tidyverse. To pull this off he uses the dplyr, ggplot2, showtext, ggtext, ggborderline, and glue packages. The functions he uses from these packages include aes, as.numeric, case_when, coord_cartesian, element_blank, element_line, element_text, factor, filter, font_add_google, geom_borderline, geom_richtext, geom_vline, ggplot, ggsave, glue, if_else, labs, library, margin, mutate, pivot_longer, rescale, scale_color_gradientn, scale_x_continuous, scale_y_continuous, select, showtext_auto, showtext_opts, theme, and tribble. The newsletter describing this visualization at a 30,000 ft view can be found [here](https://shop.riffomonas.org/posts/visualizing-the-glass-ceiling-for-women-s-history-month). You can find the original article in [The Economist](https://www.economist.com/graphic-detail/2025/03/05/the-best-places-to-be-a-working-woman-in-2025) and a [free archived version here](https://archive.is/009Rb).

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

```R
library(tidyverse)
library(showtext)
library(ggtext)
library(ggborderline)
library(glue)

font_add_google("Roboto Condensed", "roboto")
showtext_opts(dpi = 300)
showtext_auto()

data <- tribble(
  ~country,~`2016`,~`2017`,~`2018`,~`2019`,~`2020`,
    ~`2021`,~`2022`,~`2023`,~`2024`,
  "Iceland",1,3,3,1,2,2,1,1,2,
  "Sweden",2,1,1,2,1,1,2,2,1,
  "Norway",3,2,2,4,4,4,4,3,4,
  "Finland",4,4,4,3,3,3,3,4,3,
  "Poland",5,7,10,8,10,9,12,7,13,
  "France",6,5,5,5,5,7,6,5,5, #tie - 5th in list
  "Denmark",7,6,7,6,6,12,9,9,9,
  "Belgium",8,8,6,7,8,6,7,8,11,
  "Hungary",9,9,9,10,14,25,24,25,25,
  "Canada",10,10,11,12,11,10,14,14,14, #tie - 15th in list,
  "New Zealand",11,12,18,11,9,8,8,13,5, #tie - 6th in list
  "Portugal",12,11,8,9,7,5,5,6,5, #tie - 6th in list
  "OECD average",13,19,19,19,18,18,19,19,19,
  "Spain",14,15,15,14,15,13,11,12,8,
  "Australia",15,16,17,16,16,14,15,10,10,
  "Slovakia",16,13,13,17,12,11,10,15,17,
  "Israel",17,14,16,20,20,23,26,26,24,
  "Italy",18,17,12,15,13,16,16,16,16,
  "Austria",19,18,14,13,17,15,13,11,12,
  "Germany",20,21,22,21,23,19,23,22,23,
  "United States",21,20,21,23,19,21,20,23,20, #tie - 21st in list
  "Greece",22,24,23,26,26,26,25,21,20, #tie - 20th in list
  "Britain",23,26,25,24,21,17,17,20,14, #tie - 14th in list,
  "Ireland",24,22,20,18,22,20,18,17,18,
  "Netherlands",25,25,26,25,25,22,21,24,22,
  "Czech Republic",26,23,24,22,24,24,22,18,26,
  "Switzerland",27,27,27,27,27,27,27,27,27,
  "Turkey",28,28,28,28,28,28,28,29,30,
  "Japan",29,29,29,29,29,29,29,28,28,
  "South Korea",30,30,30,30,30,30,30,30,29
) %>%
  mutate(country = factor(country, levels = country),
       final_rank = if_else(country == "OECD average", NA, `2024`)) %>%
  pivot_longer(-c(country, final_rank), names_to = "year", values_to = "rank") %>%
  mutate(year = as.numeric(year))
  
label_data <- data %>%
  select(-final_rank) %>%
  filter(year == 2016 | year == 2024) %>%
  mutate(hjust = if_else(year == 2016, 1, 0),
         nudge = if_else(year == 2016, -0.05, 0.05),
         label = case_when(country == "OECD average" ~ "**OECD average**",
                           year == 2016 & rank < 13 ~ glue("{rank}\\. {country}"),
                           year == 2024 & rank < 19 ~ glue("{rank}\\. {country}"),
                           TRUE ~ glue("{rank-1}\\. {country}")),
         rank = case_when(year == 2024 & country == "New Zealand" ~ 6,
                          year == 2024 & country == "Portugal" ~ 7,
                          year == 2024 & country == "Canada" ~ 15,
                          year == 2024 & country == "United States" ~ 21,
                          TRUE ~ rank))


data %>%
  ggplot(aes(x = year, y = rank, color = final_rank, group = country)) +
  geom_borderline(linewidth = 2.25,
                  lineend = "round",
                  borderwidth = 0.25,
                  bordercolor = "white", 
                  show.legend = FALSE) +
  # geom_vline(xintercept = c(2015.9, 2024.1),
  #            linewidth = 3,
  #            color = "white") +
  geom_richtext(data = label_data,
                aes(x = year, y = rank, label = label, hjust = hjust),
                nudge_x = label_data$nudge, family = "roboto",
                size = 3.25, label.size = NA, fill = NA,
                label.margin = margin(0,0,0,0),
                inherit.aes = FALSE) +
  coord_cartesian(expand = FALSE, clip = "off") +
  scale_y_continuous(transform = "reverse",
                     limits = c(31, 0)) +
  scale_x_continuous(breaks = c(2016, 2018, 2020, 2022, 2024),
                     labels = c(2016,   18,   20,   22,   24)) +
  scale_color_gradientn(colors = c("#E4140B", "gray70", "#4D62CF"),
                        values = scales::rescale(c(30, 19, 1)),
                        na.value = "gray30") +
  labs(title = "Index rank out of 29 countries",
       x = NULL,
       y = NULL) +
  theme(
    text = element_text(family = "roboto"),
    plot.title = element_text(face = "bold", size = 10.5, hjust = 0.5,
                              margin = margin(b = 10)),
    axis.ticks = element_blank(),
    axis.text.y = element_blank(),
    axis.text.x = element_text(color = "black"),
    axis.line.x = element_line(linewidth = 0.25),
    panel.background = element_blank(),
    panel.grid.minor = element_blank(),
    panel.grid.major.y = element_blank(),
    panel.grid.major.x = element_line(color = "gray", linewidth = 0.2),
    plot.margin = margin(t = 5, r = 80, b = 5, l = 75)
  )

ggsave("glass_ceiling_index.png", width = 6, height = 5.49)
``` 