---
layout: post
title: "Using the ggridges R package to recreate an infographic describing the baby boom (CC359)"
blurb: "Livestream!"
author: "PD Schloss"
date: 2025-06-11 8:00
comments: false
youtube: HjxMDKKExhA
start_hash: 
end_hash: 
---

In this livestream, Pat recreates a ridgeline plot using the ggridges package from ggplot2 to recreate an infographic style plot showing evidence of the US Baby Boom . The plot was generated using functions from the tidyverse. To pull this off he uses the dplyr, ggplot2, showtext, and ggtext packages. The functions he uses from these packages include aes, annotate, arrow, as.double, coord_cartesian, drop_na, eq, filter, format, geom_ridgeline_gradient, ggplot, labs, max, mutate, rename_all, scale_fill_gradient, scale_x_continuous, scale_y_continuous, str_replace, theme, unit, element_blank, element_line, element_markdown, element_rect, element_text, font_add_google, ggsave, library, margin, read_table, showtext_auto, showtext_opts, and unit. The newsletter describing this visualization at a 30,000 ft view can be found [here](https://shop.riffomonas.org/posts/plotting-the-baby-boom-with-a-ridgeline-plot). You can find the original article [here](https://ourworldindata.org/baby-boom-seven-charts). The data he uses can be found [here](https://www.humanfertility.org/Country/Country?cntr=USA). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

```R
library(tidyverse)
library(showtext)
library(ggtext)
library(ggridges)

font_add_google("Playfair Display", "playfair")
font_add_google("Lato", "lato")
showtext_opts(dpi = 300)
showtext_auto()

# https://www.humanfertility.org/File/GetDocument/Files/USA/20250425/USAasfrVH.txt



read_table("USAasfrVH.txt", skip = 2, na = ".") %>%
  mutate(Age = str_replace(Age, "[-+]", "") %>% as.double()) %>%
  rename_all(tolower) %>%
  filter(cohort >= 1918 & cohort <= 1970) %>%
  drop_na(asfr) %>%
  mutate(scale_asfr = asfr / max(asfr)) %>%
  ggplot(aes(x = age, y = cohort - 0.095, height = scale_asfr, fill = asfr,
             group = cohort)) +
  geom_ridgeline_gradient(color = "#1D1E34", linewidth = 0.3, scale = 7) +
  annotate(geom = "segment", x = 10, y = 1970.5, yend = 1920,
           color = "#556680", linewidth = 0.2) +
  annotate(geom = "text",
           x = c(30, 5),
           y = c(1909.5, 1915),
           label = c("Women's age", "Women born\nin the year"),
           color = "#9CA8BB", hjust = c(0.5, 0), lineheight = 1.0,
           family = "lato", fontface = "bold", size = 8, size.unit = "pt") +
  annotate(geom = "text",
           x = 47,
           y= 1917.5,
           hjust = 0, vjust = 1,
           label = "Baby boom", color = "#9CA8BB",
           family = "playfair",
           fontface = "bold",
           size = 14, size.unit = "pt") +
  annotate(geom = "text",
           x = c(47, 47, 46, 48),
           y = c(1921, 1929, 1936, 1949),
           label = c("Women had more children\nduring the baby boom, and\nearlier in their lifetimes\nthan previous cohorts had.",
                     "The pronounced surge in\nbirths corresponds to the\nages of women in 1946.",
                     "After the baby boom, the age\nrange at childbirth narrowed;\nthe next cohorts of women\nmostly had children in their\ntwenties and early thirties.",
                     "Later cohorts of women\nhave had children across a\nmuch wider range of ages."),
           hjust = 0, vjust = 1, lineheight = 1,
           family = "lato", size = 8, size.unit = "pt", color = "#9CA8BB") +
  annotate(geom = "text",
           x = 55.5,
           y = 1962,
           label = "A value of 0.25\nmeans that 1 in 4\nof all women at\nthat age gave\nbirth that year",
           size = 6.5, size.unit = "pt", family = "lato", hjust = 0, vjust = 1,
           color = "#9CA8BB", lineheight = 1.05) +
  annotate(geom = "curve",
           x = 55, xend = 57, y = 1960.5, yend = 1961.5, color = "#9CA8BB",
           curvature = -0.5, angle = 90, linewidth = 0.2,
           arrow = arrow(type = "closed", angle = 20, length = unit(3, "pt"))) +
  coord_cartesian(ylim = c(1970.5, 1914),
                  xlim = c(10, 48),
                  expand = FALSE, clip = "off") +
  scale_y_continuous(breaks = seq(1920, 1970, 5),
                     transform = "reverse") +
  scale_x_continuous(breaks = c(20, 30, 40),
                     labels = c(20, 30, 40), position = "top") +
  scale_fill_gradient(low = "#303D5A", high = "#8DA3B8",
                      breaks = seq(0, 0.25, 0.05),
                      labels = c("0",
                                 seq(0.05, 0.25, 0.05) %>% format(nsmall=2))) +
  labs(x = NULL, y = NULL, 
       title = "The US Baby Boom: When did each cohort of\nwomen have children?",
       subtitle = "The age-specific fertility rate is shown by the height and color of the curves. It is the number of\nbirths per woman of that particular age.",
       caption = "**Data source:** Human Fertility Database (2024)<br>**OurWorldinData.org** - Research and data to make progress against<br>the world's largest problems.",
       fill = "Age-specific fertility rate") +
  theme(
    plot.background = element_rect(fill = "#1D1E34", color = "#1D1E34"),
    plot.title.position = "plot",
    plot.title = element_text(family = "playfair", color = "#E9BF37",
                              face = "bold", lineheight = 1.1),
    plot.subtitle = element_text(color = "#9CA8BB", family = "lato",
                                 size = 8, margin = margin(b = 18)),
    plot.caption.position = "plot",
    plot.caption = element_markdown(hjust = 0, color = "#74869F", size = 6.5, 
                                    family = "lato", lineheight = 1.2,
                                    margin = margin(t = 12)),
    plot.margin = margin(t = 13, r = 12, b = 8, l = 5),
    panel.background = element_blank(),
    panel.grid.major = element_line(color = "#556680", linewidth = 0.2),
    panel.grid.minor = element_blank(),
    axis.ticks = element_line(color = "#556680", linewidth = 0.2),
    axis.text = element_text(color = "#9CA8BB", family = "lato", size = 7.5),
    axis.text.x.top = element_text(margin = margin(b = 6)),
    axis.text.y = element_text(margin = margin(l = 3, r = 6)),
    legend.justification.right = "bottom",
    legend.text = element_text(color = "#9CA8BB", family = "lato", size = 6.5,
                               margin = margin(l = 3)),
    legend.title = element_text(color = "#9CA8BB", family = "lato", size = 6.5),
    legend.background = element_blank(),
    legend.key.width = unit(10, "pt"),
    legend.key.height = unit(10.2, "pt"),
    legend.margin = margin(0, 0, 3, 0),
    legend.ticks = element_blank()
  ) -> p

# this is needed to add the logo to the upper right corner of plot
# 
# library(cowplot)
# ggdraw(p) +
#   draw_image("logo.png", x = 1, y = 1.06,
#             hjust = 1, vjust = 1, width = 0.1, height = 0.2)

ggsave("fertility_ridgeline.png", width = 4.66, height = 6)
```
