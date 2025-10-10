---
layout: post
title: "Using ggplot2 to make a faceted bar plot that you can't tell has facets (CC370)"
blurb: "Trying out the new margin_auto ggplot2 function"
author: "PD Schloss"
date: 2025-09-24 9:00
comments: false
youtube: DWkBiSiiwLI
start_hash: 
end_hash: 
---

Pat recreates a figure from the New York Times that has a series of titled bar plots that are part of a single figure. He shows a variety of approaches to create this appearance before settling in on using facet_wrap. Along the way, he also tries out some new features from the recently released version 4 of ggplot2. He recreated the NY Times figure using R, ggplot2, showtext, ggtext, and other tools from the tidyverse. The functions I used from these packages include aes, coord_cartesian, element_blank, element_text, element_textbox_simple, facet_wrap, factor, font_add_google, geom_col, geom_text, ggplot, ggsave, labs, library, margin, margin_auto, mutate, paste0, scale_fill_manual, showtext_auto, showtext_opts, theme, tribble, unique, and unit. You can find the code he developed in this episode [here](https://www.riffomonas.org/code_club/2025-09-24-policy-wording). The newsletter describing this visualization at a 30,000 ft view can be [found here](https://shop.riffomonas.org/posts/visualize-this-if-you-support-something-do-you-actually-want-it-doney). You can find the original article presenting the figure [here for free](https://archive.is/crTI0). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```R
library(tidyverse)
library(showtext)
library(ggtext)

font_add_google("Libre Franklin", "franklin",
                regular.wt = 500, bold.wt = 800)
font_add_google("Noto Serif", "noto_serif", bold.wt = 700)
showtext_opts(dpi = 300)
showtext_auto()

tribble(
  ~policy, ~choice, ~percentage,
  "Add 5,000 police officers", "Support", 73,
  "Add 5,000 police officers", "N.Y.C. should do this", 63,
  "Limit rent stabilization by income", "Support", 71,
  "Limit rent stabilization by income", "N.Y.C. should do this", 61,
  "Freeze prices on rent-stabilized units", "Support", 70,
  "Freeze prices on rent-stabilized units", "N.Y.C. should do this", 66,
  "Increase taxes on the wealthiest", "Support", 68,
  "Increase taxes on the wealthiest", "N.Y.C. should do this", 69,
  "Make buses free", "Support", 60,
  "Make buses free", "N.Y.C. should do this", 44) %>%
  mutate(policy = factor(policy, levels = unique(policy))) %>%
  ggplot(aes(x = percentage, y = choice, fill = choice)) +
  geom_col(show.legend = FALSE) +
  # option 2 to put choice within bar
  geom_text(aes(label = choice), x = 1, hjust = 0, color = "gray30",
            family = "franklin", size = 9.5, size.unit = "pt") +
  geom_text(aes(label = paste0(percentage, "%")),
            hjust = 0, nudge_x = 1, color = "gray30",
            family = "franklin", size = 9.5, size.unit = "pt") +
  facet_wrap(~policy, ncol = 1) +
  coord_cartesian(clip = "off", expand = FALSE,
                  xlim = c(0, 100)) +
  scale_fill_manual(
    values = c("#A8BEB3", "#F7A480")
  ) +
  labs(
    title = "Wording affects policy popularity",
    subtitle = "Percentage of respondents who **<span style='color:#E7855F'>support a policy</span>** versus the percentage who think that<br><b><span style='color:#96A79F;'>New York City should</span></b> do the policy",
    caption = "Note: \"Limit rent stabilization by income\" refers to limiting rent-stabilized apartments to those who would be spending at least 30 percent of their income on rent. • Source: New York Times/Siena poll conducted Sept. 2-6, 2025"
  ) +
  theme(
    text = element_text(family = "franklin"),
    strip.text = element_text(face = "bold", hjust = 0, size = 10,
                              margin = margin_auto(4, 0)),
    strip.background = element_blank(),
    panel.grid = element_blank(),
    panel.background = element_blank(),
    # option 1 to put choice within bar
    # axis.text.y = element_text(hjust = 0, margin = margin(l = 3, r = -100))
    axis.title = element_blank(),
    axis.text = element_blank(),
    axis.ticks = element_blank(),
    plot.caption = element_textbox_simple(size = 8, color = "gray40",
                                          lineheight = 1.3,
                                          margin = margin(t = 20, b = 10)),
    plot.subtitle = element_textbox_simple(size = 9, lineheight = 1.5,
                                           margin = margin(t = 7, b = 13)),
    plot.title = element_text(face = "bold", family = "noto_serif"),
    plot.margin = margin_auto(10, 10),
    panel.spacing.y = unit(13, "pt")
  )

ggsave("policy_wording.png", width = 6, height = 6.22)
```