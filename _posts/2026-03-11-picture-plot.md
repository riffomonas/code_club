---
layout: post
title: "Using ggplot2 to recreate a figure made up of pictures (CC406)"
blurb: "Same figure, two appraoches"
author: "PD Schloss"
date: 2026-03-11 12:00
comments: false
youtube: 0P4sIpgkbIw
start_hash: 
end_hash: 
---

In this livestream, Pat refactors a panel of images from a paper recently published in Nature using two approaches that leverage geom_richtext from the ggtext package. One uses facet_grid from the ggh4x helper package and the other uses facet_wrap from ggplot2 without the helper package. He created the figures using R, dplyr, ggplot2, ggtext, glue, ggh4x, and other tools from the tidyverse. The functions he used from these packages included aes, annotate, coord_cartesian, element_blank, element_line, element_markdown, element_rect, element_text, expand.grid, facet_nested, facet_wrap, factor, geom_richtext, ggplot, ggsave, glue, labs, library, margin, mutate, rev, scale_x_discrete, scale_y_discrete, theme, and unit. The original Nature article can be [found here](https://www.nature.com/articles/s41586-026-10207-1). A video critiquing the original version of the figure can be [found here](https://youtu.be/WWMQuTwqRVA). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```R
library(tidyverse)
library(glue)
library(ggtext)
library(ggh4x) #facet_nested / facet_grid2

time_vector <- c(0, 50, 130)
microscopy_vector <- c("phase", "cfp")
construct_vector <- c("ev", "snipe")

img_size <- 52

# approach using ggh4x

expand.grid(time = time_vector, microscopy = microscopy_vector,
            construct = construct_vector) %>%
  mutate(png = glue("images/{construct}_{microscopy}_{time}.png"),
         img = glue("<img src = '{png}' 
                    height = '{img_size}' width = '{img_size}' />"),
         time = factor(time, levels = time_vector,
                       labels = c("0 min", "50 min", "130 min")),
         microscopy = factor(microscopy, levels = microscopy_vector,
                             labels = c("Phase", "CFP-ParB")),
         construct = factor(construct, levels = construct_vector,
                            labels = c("&lambda;<sup><i>parS</i></sup> lysogen<br>+ EV",
                                       "&lambda;<sup><i>parS</i></sup> lysogen<br>+ SNIPE"))) %>%
  ggplot(aes(x = 0, y = 0, label = img)) +
  geom_richtext(label.padding = unit(c(0, 0, -2.2, 0), "pt"),
                label.r = unit(0, "pt"), label.color = "black",
                label.size = 0.75) +
  facet_nested(time ~ construct + microscopy,
               switch = "y",
               nest_line = element_line()) +
  coord_cartesian(expand = FALSE, clip = "off") +
  labs(
    x = NULL,
    y = "Time after induction"
  ) +
  theme(
    plot.margin = margin(5, 3, 4, 5),
    panel.spacing.x = unit(c(3, 18, 3), "pt"),
    panel.spacing.y = unit(1, "pt"),
    panel.grid = element_blank(),
    axis.text = element_blank(),
    axis.ticks = element_blank(),
    axis.title.y = element_text(size = 8),
    
    strip.text.y.left = element_text(angle = 0, hjust = 1, size = 8,
                                     margin = margin(l = 4, r = 2)),
    strip.text.x = element_markdown(size = 8, lineheight = 1.1,
                                    margin = margin(0, 0, 2, 0)),
    strip.background = element_blank()
  )

ggsave("picture_plot_ggh4x.png", width = 4, height = 2.96)



# ggplot2 without ggh4x

expand.grid(time = time_vector, microscopy = microscopy_vector,
            construct = construct_vector) %>%
  mutate(png = glue("images/{construct}_{microscopy}_{time}.png"),
         img = glue("<img src = '{png}' 
                    height = '{img_size}' width = '{img_size}' />"),
         time = factor(time, levels = rev(time_vector),
                       labels = rev(c("0 min", "50 min", "130 min"))),
         microscopy = factor(microscopy, levels = microscopy_vector,
                             labels = c("Phase", "CFP-ParB")),
         construct = factor(construct, levels = construct_vector,
                            labels = c("&lambda;<sup><i>parS</i></sup> lysogen<br>+ EV",
                                       "&lambda;<sup><i>parS</i></sup> lysogen<br>+ SNIPE"))) %>%
  ggplot(aes(x = microscopy, y = time, label = img)) +
  geom_richtext(label.padding = unit(c(0, 0, -2.2, 0), "pt"),
                label.size = 0.75, label.r = unit(0, "pt")) +
  facet_wrap(~construct) +
  annotate(
    geom = "segment",
    x = 0.5, xend = 2.5, y = 3.75,
    linewidth = 0.3
  ) +
  scale_x_discrete(position = "top", expand = expansion(add = 0.48)) +
  scale_y_discrete(expand = expansion(add = 0.5)) +
  coord_cartesian(clip = "off",
                  ylim = c(1, 3)) +
  labs(
    x = NULL,
    y = "Time after induction"
  ) +
  theme(
    plot.margin = margin(6, 3, 4, 5),
    strip.placement = "outside",
    strip.text.x = element_markdown(size = 8, lineheight = 1.1,
                                    margin = margin(2, 0, 3, 0)),
    strip.background.x = element_blank(),
    axis.title = element_text(size = 8),
    axis.text = element_text(size = 8, color = "black"),
    axis.text.x.top = element_text(margin = margin(t = 1.5, b = 3)),
    axis.ticks = element_blank(),
    panel.spacing.x = unit(20, "pt"),
    panel.background = element_blank(),
    panel.grid = element_blank()
  )

ggsave("picture_plot_ggplot2.png", width = 4, height = 2.96)
```