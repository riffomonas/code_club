---
layout: post
title: "Recreating graphic from the FT showing the year to date change in the US Dollar Index (CC377)"
blurb: "How do we recreate this plate of data spaghetti?"
author: "PD Schloss"
date: 2025-10-29 12:00
comments: false
youtube: ccxlUJRTyjk
start_hash: 
end_hash: 
---

Pat recreates a line plot from the Financial Times that shows the year to date change in the US Dollar Index (DXY) relative to other years in this livestream. He recreated the figure using R, ggplot2, showtext, ggborderline, and other tools from the tidyverse. The functions he used from these packages included aes, annotation_custom, arrange, coord_cartesian, element_line, element_rect, element_text, filter, font_add_google, geom_borderline, geom_hline, geom_segment, geom_text, ggplot, ggsave, labs, library, margin, mdy, mutate, parse_date, rasterGrob, readPNG, read_csv, rename_all, rsvg_png, scale_color_manual, scale_y_continuous, showtext_auto, showtext_opts, slice_max, theme, today, tolower, unit, update, year, and ymd. The newsletter describing this visualization at a 30,000 ft view can be found [here](https://shop.riffomonas.org/posts/how-bad-has-2025-been-for-the-us-dollar-is-this-picture-worth-1000-words). You can find the original article presenting the figure [here]( https://www.ft.com/content/ac9e7ee1-ebe5-431a-a315-b833de728ec9). [Here's a video](https://youtu.be/lsD2_lxE18g) critiquing the original plot. If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

```R
library(tidyverse)
library(ggborderline)
library(showtext)
library(rsvg)

logo_svg <- "https://public.flourish.studio/uploads/43d2c31d-7939-4110-a079-53f54ba93ac6.svg"
rsvg_png(logo_svg, "ft_logo.png", width = 2000)

get_png <- function(filename) {
  grid::rasterGrob(png::readPNG(filename), interpolate = TRUE)
}

l <- get_png("ft_logo.png")


font_add_google("Noto Sans", "noto", bold.wt = 600)
showtext_opts(dpi = 300)
showtext_auto()

my_heighshowtextmy_height <- 6 / 2162 * 1518


all_dollar <- read_csv("HistoricalPrices.csv") %>%
  rename_all(tolower) %>%
  mutate(date = mdy(date), #parse_date(date, format = "%m/%d/%y")
         year = year(date),
         fake_date = update(date, year = 2024),
         this_year = year == year(today())) %>%
  arrange(date) %>%
  mutate(rel_change = 100 * (close - open[1]) / open[1], .by = year) %>%
  filter(
    year > 1990 & fake_date <= "2024-09-18"
  )

select_dollar <- all_dollar %>%
  filter(year %in% c(1991, 2013, year(today()))) %>% 
  slice_max(date, by = "year")

all_dollar %>%
  ggplot(aes(x = fake_date, y = rel_change, group = year, color = this_year)) +
  geom_borderline(
    show.legend = FALSE, bordercolor = "#FFF2E5",
    linewidth = 0.6) +
  geom_text(
    show.legend = FALSE,
    data = select_dollar,
    aes(y = rel_change, label = year, color = this_year),
    x = ymd("2024-09-28"),
    family = "noto", size = 8.5, size.unit = "pt", fontface = "bold") +
  geom_segment(
    show.legend = FALSE,
    data = select_dollar,
    aes(y = rel_change),
    x = ymd("2024-09-18"), xend = ymd("2024-09-20"),
    color = "#999189", linewidth = 0.3,
  ) +
  geom_hline(
    yintercept = 0, linetype = "11"
  ) +
  scale_color_manual(
    breaks = c(FALSE, TRUE),
    values = c("#D9CDC3", "#F34E5B")
  ) +
  scale_y_continuous(
    breaks = seq(-10, 15, 5)
  ) +
  coord_cartesian(
    ylim = c(-11.5, NA),
    expand = FALSE, clip = "off"
  ) +
  labs(
    title = "Dollar's worst year in decades",
    subtitle = "% change in the US Dollar index for each year up until September 18",
    caption = "Source: LSEG, FT calculations"
  ) +
  theme(
    text = element_text(family = "noto"),
    plot.background = element_rect(fill = "#FFF2E5", color = "#FFF2E5"),
    plot.margin = margin(t = 7, r = 35, b = 7, l = 7),
    plot.title.position = "plot",
    plot.title = element_text(size = 12),
    plot.subtitle = element_text(size = 9, margin = margin(t = 11, b = 22)),
    plot.caption.position = "plot",
    plot.caption = element_text(margin = margin(r = -27, t = 22,), size = 7),
    panel.background = element_blank(),
    panel.grid.minor = element_blank(),
    panel.grid.major.x = element_blank(),
    panel.grid.major.y = element_line(color = "#D9CDC3", linewidth = 0.3),
    axis.title = element_blank(),
    axis.ticks.y = element_blank(),
    axis.text.y = element_text(margin = margin(r = 8), color = "black"),
    axis.ticks.x = element_line(color = "#999189", linewidth = 0.3),
    axis.ticks.length.x = unit(5, "pt"),
    axis.text.x = element_text(margin = margin(t = 4))
  ) +
  annotation_custom(l, xmin = ymd("2023-12-16"), xmax = ymd("2024-02-25"),
                    ymax = -24)

ggsave("dollar_year.png", width = 6, height = my_height)


all_dollar %>%
  slice_max(date, by = year) %>%
  arrange(rel_change)
```