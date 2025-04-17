---
layout: post
title: "Scraping data from the web in R using rvest to create NY Times plot of March Madness Championship Game viewership (CC354)"
blurb: "March Madness!!!"
author: "PD Schloss"
date: 2025-04-03 8:00
comments: false
youtube: UPE7Z6Gfhno
start_hash: 
end_hash: 
---

Pat uses R to recreate a plot from the New York Times showing the tradjectory of viewership of the men's and women's Basketball NCAA Championship game using rvest and functions from the tidyverse. To pull this off he uses the rvest, dplyr, ggplot2, showtext, and glue packages. The functions he uses from these packages include aes, annotate, arrange, as.numeric, bind_rows, coord_cartesian, element_blank, element_line, element_text, filter, font_add_google, full_join, geom_line, geom_point, geom_text, ggplot, ggsave, glue, html_elements, html_node, html_nodes, html_table, html_text, if_else, labs, library, map_dfr, margin, mutate, na_if, pivot_longer, read_html, scale_color_manual, scale_x_continuous, scale_y_continuous, select, seq, showtext_auto, showtext_opts, str_replace, str_replace_all, str_subset, theme, tibble, and unit. The newsletter describing this visualization at a 30,000 ft view can be found [here](https://shop.riffomonas.org/posts/visualizing-men-s-and-women-s-march-madness-with-ggplot2-and-rvest). You can find the original article [here](https://www.nytimes.com/2025/03/16/briefing/womens-basketball.html). The women's data can be found at [here](https://www.nielsen.com/news-center/2024/womens-college-basketball-championship-draws-record-breaking-18-9-million-viewers/) and the men's data [here](https://www.sportsmediawatch.com/ncaa-final-four-ratings-history-most-watched-games-cbs-tbs-nbc/). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

```R
library(tidyverse)
library(showtext)
library(rvest)
library(glue)

font_add_google("Libre Franklin", "franklin")
font_add_google("Domine", "domine")
showtext_opts(dpi = 300)
showtext_auto()

womens_page <- "https://www.nielsen.com/news-center/2024/womens-college-basketball-championship-draws-record-breaking-18-9-million-viewers/"

womens_viewership <- read_html(womens_page) %>%
  html_node("tbody") %>%
  html_table() %>%
  select(year = X1, viewership = X6) %>%
  mutate(viewership = str_replace_all(viewership, ",", ""),
         women = viewership %>% as.numeric() / 1000000) %>%
  select(-viewership)


mens_page <- "https://www.sportsmediawatch.com/ncaa-final-four-ratings-history-most-watched-games-cbs-tbs-nbc/"

mens_years <- read_html(mens_page) %>%
  html_elements("h4") %>%
  html_text() %>%
  str_subset("^\\d{4}$")

mens_viewership <- read_html(mens_page) %>%
  html_nodes("table") %>%
  map_dfr(~html_table(.x)) %>%
  filter(Window == "Champ.") %>%
  mutate(year = mens_years) %>%
  select(year, viewership = Vwrs) %>%
  mutate(viewership = str_replace(viewership, "M\n?.*", ""),
         viewership = na_if(viewership, "n.a."),
         men = as.numeric(viewership),
         year = as.numeric(year)) %>%
  select(-viewership)

full_data <- full_join(womens_viewership, mens_viewership, by = "year") %>%
  pivot_longer(-year, names_to = "tournament", values_to = "view_mil") %>%
  bind_rows(tibble(year = 2020,
                   tournament = c("women", "men"),
                   view_mil = c(NA, NA))) %>%
  filter(year >= 1995) %>%
  arrange(-year)

label_data <- full_data %>%
  filter(year == 2024) %>%
  mutate(label = if_else(tournament == "women",
                         glue("Women's:\n{round(view_mil, 1)} million"),
                         glue("Men's:\n{round(view_mil, 1)} million")))


full_data %>%
  ggplot(aes(x = year, y = view_mil, color = tournament, group = tournament)) +
  geom_line(linewidth = 0.75, show.legend = FALSE) +
  geom_text(data = label_data, show.legend = FALSE,
            aes(x = year + 1, y = view_mil, color = tournament, label = label),
            hjust = 0, family = "franklin", fontface = "bold",
            size = 9, size.unit = "pt", lineheight = 1,
            vjust = c(0.8, 0.9)) +
  geom_point(data = label_data, show.legend = FALSE,
             aes(x = year, y = view_mil, color = tournament)) +
  annotate(geom = "text", hjust = 0, family = "franklin", color = "gray40",
           x = 1992.1,
           y = c(seq(5, 20, 5) + 0.6, 25),
           label = c(seq(5, 20, 5), "25 million\nviewers"),
           size = 8.5, size.unit = "pt", lineheight = 1) +
  annotate(geom = "text", hjust = 0, vjust = 0,
           family = "franklin", color = "black",
           size = 9, size.unit = "pt", 
           x = 2025, y = 20, label = "2024 finals") +
  labs(x = NULL, y = NULL,
       title = "N.C.A.A. basketball championship viewers, 1995-2024",
       caption = "Source: Nielsen • By The New York Times / Pat Schloss") +
  scale_y_continuous(breaks = seq(0, 30, 5)) +
  scale_x_continuous(breaks = seq(1995, 2025, 5)) +
  coord_cartesian(ylim = c(0, NA),
                  xlim = c(1992, 2024.5),
                  expand = FALSE, clip = "off") +
  scale_color_manual(breaks = c("men", "women"),
                     values = c("#999999", "#E57D01")) +
  theme(
    text = element_text(family = "franklin"),
    plot.title = element_text(family = "domine", face = "bold", size = 12.5,
                              margin = margin(t = 3, b = 14)),
    plot.caption = element_text(hjust = 0, color = "gray40", size = 8,
                                margin = margin(t = 13)),
    axis.text.y = element_blank(),
    axis.text.x = element_text(color = "gray40", margin = margin(t = 4)),
    axis.ticks.y = element_blank(),
    axis.ticks.length.x = unit(4, "pt"),
    axis.ticks.x = element_line(linewidth = 0.2, color = "gray70"),
    panel.grid.minor = element_blank(),
    panel.grid.major.x = element_blank(),
    panel.grid.major.y = element_line(color = "gray70", linewidth = 0.2),
    panel.background = element_blank(),
    plot.margin = margin(t = 5, r = 75, b = 5, l = 5)
  )

ggsave("ncaa_final.png", width = 6, height = 4.4)
```
