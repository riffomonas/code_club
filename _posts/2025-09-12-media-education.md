---
layout: post
title: "Trying Positron for the first time by remaking a plot from Pew (CC369)"
blurb: "Cool, but some kinks to be worked out"
author: "PD Schloss"
date: 2025-09-12 9:00
comments: false
youtube: IP9jpxyoW5U
start_hash: 
end_hash: 
---

The first public release of Positron is out! The company that makes RStudio, Posit, now has released a second integrated development environment (IDE). They promise to continue supporting both RStudio and Positron. I haven't looked at Positron in a while and thought it would be fun to try out the new version by making a bar plot that was released by the Pew Research Center. Along the way I discovered some glitches that I'll have to explore in the future. I recreated the figure from Pew using R, ggplot2, showtext, ggtext, and other tools from the tidyverse. The functions I used from these packages include annotate, coord_cartesian, labs, scale_color_identity, theme, aes, aes, as.character, element_blank, element_markdown, element_text, factor, font_add_google, geom_col, geom_point, geom_text, ggplot, ggsave, if_else, library, margin, mutate, paste0, rev, showtext_auto, showtext_opts, and tribble. The newsletter describing this visualization at a 30,000 ft view can be [found here](https://shop.riffomonas.org/posts/thinking-about-how-to-extract-and-visualize-data-from-a-pdf-why). You can find the original article presenting the [figure here](https://www.pewresearch.org/short-reads/2025/08/18/how-the-audiences-of-30-major-news-sources-differ-in-their-levels-of-education/). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!


```R
library(tidyverse)
library(showtext)
library(ggtext)

font_add_google("Libre Franklin", family = "libre", regular.wt = 500, bold.wt = 800)
showtext_opts(dpi = 300)
showtext_auto()

tribble(
  ~source, ~college_plus, ~some_college, ~no_college,
  "The Atlantic", 62, 23, 14,
  "Axios", 59, 21, 19,
  "NPR", 58, 27, 14,
  "The New York Times", 56, 24, 19,
  "The Guardian", 53, 28, 18,
  "The Wall Street Journal", 53, 26, 21,
  "Politico", 52, 27, 20,
  "The Associated Press", 52, 28, 20,
  "BBC News", 49, 27, 23,
  "The Washington Post", 47, 30, 22,
  "HuffPost", 47, 28, 25,
  "The Hill", 45, 25, 29,
  "Forbes", 42, 30, 28,
  "PBS", 41, 29, 30,
  "CNN", 38, 28, 33,
  "New York Post", 38, 33, 29,
  "The Daily Wire", 37, 37, 26,
  "MSNBC", 37, 30, 33,
  "USA Today", 34, 30, 36,
  "Breitbart", 34, 32, 34,
  "Newsweek", 32, 32, 35,
  "NBC News", 32, 29, 39,
  "ABC News", 31, 30, 39,
  "CBS News", 30, 30, 40,
  "Tucker Carlson Network", 28, 33, 40,
  "Newsmax", 28, 35, 37,
  "Fox News", 27, 30, 42,
  "The Joe Rogan Experience", 27, 35, 38,
  "Telemundo", 16, 26, 56,
  "Univision", 15, 26, 57) %>%
  mutate(
    source = factor(source, levels = rev(source)),
    x_label = if_else(college_plus > 34, college_plus + 1.5, college_plus - 5),
    color_label = if_else(college_plus > 34, "black", "white"),
    label = if_else(
      source == "The Atlantic",
      paste0(college_plus, "%"),
      as.character(college_plus))
    )  %>%
  ggplot(aes(x = college_plus, y = source, label = label)) +
  geom_col(fill = "#733E46", width = 0.8) +
  geom_point(x = 36, color = "#D1A8AF", size = 3) +
  geom_text(
    aes(x = x_label, color = color_label),
    hjust = 0, family = "libre") +
  annotate(
    geom = "text",
    x = c(40, 61),
    y = 31,
    label = c(
      "36% of U.S. adults are\ncollege graduates.",
      "62% of people who\nregularly get news\nfrom The Atlantic are\ncollege graduates."),
    vjust = 0, hjust = c(1, 0.5), family = "libre"
  ) +
  annotate(
    geom = "point",
    x = c(36, 62),
    y = 30.75,
    shape = 25, fill = c("#D1A8AF", "#733E46"), color = c("#D1A8AF", "#733E46")
  ) +
  scale_color_identity() +
  coord_cartesian(
    expand = FALSE, clip = "off",
    xlim = c(0, 75), ylim = c(0.5, 33.5)) +
  labs(
    title = "Across U.S. audiences of 30 major news sources, the\nshare with a college degree varies widely",
    subtitle = "Among U.S. adults who regularly get news from ___,% who have a\nbachelor's degree or more education",
    caption = "Source: Pew Research Center survey of 9,482 U.S. adults conducted March 10-16, 2025;<br>the U.S. adult figure is from Pew Research Center analysis of data from the 2024 Current<br>Population Survey, Annual Social and Economic Supplement (PUMS).<br>**<span style = 'color:black;'>PEW RESEARCH CENTER</span>**",
    x = NULL, y = NULL
  ) +
  theme(
    text = element_text(family = "libre"),
    plot.title.position = "plot",
    plot.caption.position = "plot",
    plot.title = element_text(face = "bold", size = 16, lineheight = 1.1),
    plot.subtitle = element_text(
      face = "italic", family = "serif", size = 13, 
      lineheight = 1.1, margin = margin(t = 5, b = 20)
    ),
    plot.caption = element_markdown(
      hjust = 0, color = "gray50", size = 10, lineheight = 1.5
    ),
    plot.margin = margin(t = 18, r = 10, b = 25),
    axis.ticks = element_blank(),
    axis.text.x = element_blank(),
    axis.text.y = element_text(
      size = 11, margin = margin(l = 15, r = 5), color = "black"
    ),
    panel.grid = element_blank(),
    panel.background = element_blank()
  )

ggsave("media_education.png", width = 6, height = 11.77)

# Issues with Positron
# * plot window won't go away
# * the help window being weird - see if I can recreate
# * resolution using element_textbox_simple
# * background color highlighting of things like "gray50"; does it for hex but not word
# * tabs after inserting line break to code
```