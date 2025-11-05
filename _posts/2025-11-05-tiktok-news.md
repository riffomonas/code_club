---
layout: post
title: "Recreating Pew Research plot showing where Americans get their news from on social media (CC379)"
blurb: "Practicing more features from latest version of ggplot2"
author: "PD Schloss"
date: 2025-10-29 12:00
comments: false
youtube: ccxlUJRTyjk
start_hash: 
end_hash: 
---

Pat recreates a line plot from the Pew Research Center that shows how often users of different social media platforms get their news there in this livestream. He recreated the figure using R, ggplot2, showtext, ggtext, and other tools from the tidyverse. The functions he used from these packages included aes, annotate, arrange, as.numeric, bind_rows, c, case_when, coord_cartesian, desc, distinct, drop_na, element_blank, element_line, element_markdown, element_text, element_textbox_simple, facet_wrap, factor, filter, font_add_google, geom_line, geom_point, geom_text, ggplot, ggsave, if_else, labs, library, map, margin, mutate, n, nest, pivot_longer, read_csv, rename, scale_x_continuous, showtext_auto, showtext_opts, slice_max, slice_min, theme, unit, and unnest. The newsletter describing this visualization at a 30,000 ft view can be [found here](https://shop.riffomonas.org/posts/i-have-news-where-do-you-get-your-news). You can find the original article presenting the figure at the [Pew Research Center's website](https://www.pewresearch.org/short-reads/2025/09/25/1-in-5-americans-now-regularly-get-news-on-tiktok-up-sharply-from-2020/). Here's a [video](https://youtu.be/J9UWsECiqeI) critiquing the original plot. If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

```R
library(tidyverse)
library(ggtext)
library(showtext)

font_add_google("Libre Franklin", "franklin")
font_add_google("Gelasio", "gelasio")
showtext_opts(dpi = 300)
showtext_auto()

width <- 6
height <- width / 1280 * 840

platform_labels <- read_csv("https://www.pewresearch.org/wp-content/uploads/sites/20/2025/09/PJ_2025.09.25_social-media-news_consumption-us-adults-data.csv") %>%
  rename(year = "% of U.S. adults who say they regularly get news on each social media site") %>%
  filter(year == 2025) %>%
  pivot_longer(-year, names_to = "platform", values_to = "percentage") %>%
  arrange(desc(percentage)) %>%
  mutate(
    order = 1:n(),
    order = case_when(platform == "Rumble" ~ 10,
                      platform == "Truth Social" ~ 11,
                      TRUE ~ order),
    pretty = case_when(platform == "TikTok" ~ "**TikTok**",
                         platform == "X/Twitter" ~ "X<br>(Twitter)",
                         platform == "Truth Social" ~ "Truth<br>Social",
                         TRUE ~ platform)
    
    ) %>%
  arrange(order)
  
social_data <- read_csv("https://www.pewresearch.org/wp-content/uploads/sites/20/2025/09/PJ_2025.09.25_social-media-news_consumption-site-users-data.csv") %>%
  rename(year = "% of each social media site’s users who say they regularly get news there") %>%
  filter(str_detect(year, "^20")) %>%
  pivot_longer(-year, names_to = "platform", values_to = "percentage") %>%
  mutate(
    year = as.numeric(year),
    platform = factor(platform,
                      levels = platform_labels$platform,
                      labels = platform_labels$pretty)
  ) %>%
  drop_na()

percentage_labels <- social_data %>%
  nest(data = -platform) %>%
  mutate(head_tail = map(
    data,
    ~distinct(bind_rows(slice_min(.x, year), slice_max(.x, year))))) %>%
  unnest(head_tail) %>%
  mutate(
    hjust = if_else(year == 2025, 0.8, 0.2),
    y_label = case_when(
      year != 2025 & platform %in% c("**TikTok**", "Nextdoor",  "Rumble") ~ percentage - 2.75,
      year == 2025 & platform %in% c("X<br>(Twitter)", "WhatsApp",  "Threads", "Truth<br>Social", "Bluesky") ~ percentage - 2.75,
      TRUE ~ percentage + 2.75
    )
  )

social_data %>%
  ggplot(aes(x = year, y = percentage, group = platform)) +
  annotate(
    geom = "rect",
    xmin = 2019.5, xmax = 2025.5,
    ymin = 1, ymax = 75, fill = "#F0ECE4", layout = 4
  ) +
  geom_line(color = "#743D45", linewidth =1.25) +
  geom_point(shape = 21, fill = "white", color = "#743D45") +
  geom_text(
    data = percentage_labels,
    aes(y = y_label, label = percentage, hjust = hjust),
    family = "franklin", size = 7.5, size.unit = "pt") +
  facet_wrap(~platform, nrow = 1) +
  annotate(
    geom = "segment",
    x = 2025.5, y = 1, yend = 75, linewidth = 0.3, linetype = "dotted",
    layout = 1:11, color = "gray") +
  scale_x_continuous(
    breaks = 2020:2025,
    labels = c("'20", "", "", "", "", "'25"),
    expand = c(0, 0)
  ) +
  coord_cartesian(
    xlim = c(2020, 2025),
    ylim = c(10, 64),
    clip = "off") +
  labs(
    title = "More than half of U.S. adult TikTok users get news there, up from 22% in 2020",
    subtitle = "*% of each social media site\'s users who **regularly** get news there*",
    caption = "Note: The other response option was \"No, don\'t regularly get news on this.\" Only respondents who indicated that they use each site were asked if they regularly get news on it. Social media sites are shown left to right in descending order by the share of U.S. adults who regularly get news there.
<br>Source: Survey of U.S. adults conducted Aug. 18-24, 2025.<br><br>
<span style = 'color: black;'>**PEW RESEARCH CENTER**</span>"
  ) +
  theme(
    text = element_text(family = "franklin"),
    
    panel.grid = element_blank(),
    panel.background = element_blank(),
    
    plot.margin = margin(t = 11, r = 8, b = 11, l = 8),
    plot.title = element_text(face = "bold", size = 10.5,
      margin = margin(l = -6)),
    plot.subtitle = element_markdown(
      family = "gelasio", size = 8.25, color = "#5A5656",
      margin = margin(t = 7, b = 7, l = -6)),
    plot.caption = element_textbox_simple(
      color = "#5A5656", size = 6.5, lineheight = 1.4,
      margin = margin(t = 11, l = -6)),
    
    strip.text = element_markdown(size = 6.5, color = "black", vjust = 1),
    strip.background = element_blank(),
    strip.clip = "off",
    
    axis.title = element_blank(),
    axis.text.y = element_blank(),
    axis.ticks.y = element_blank(),
    axis.line.x = element_line(color = "gray", linewidth = 0.3),
    axis.ticks.x = element_line(color = "gray", linewidth = 0.3),
    axis.ticks.length.x = unit(4, "pt"),
    axis.text.x = element_text(
      size = 6.25, hjust = c(0.2, 0, 0, 0, 0, 0.8),
      margin = margin(t = 3))
  )

ggsave(file = "tiktok_news.png", width = width, height = height)
```