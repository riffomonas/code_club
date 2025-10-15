---
layout: post
title: "Using Visual Studio Code to recreate a New York Times slope plot with R's ggplot2 (CC373)"
blurb: "Pat's favorite type of figure"
author: "PD Schloss"
date: 2025-10-10 9:00
comments: false
youtube: AYpjjvuOE_o
start_hash: 
end_hash: 
---

Pat uses the R packages in Virtual Studio Code to recreate a slope plot from the New York Times that shows the change in sentiment about the role of Israel in the number of civilian deaths in Gaza between 2023 and 2025. He recreated the figure using R, ggplot2, showtext, ggtext, and other tools from the tidyverse. The functions he used from these packages included aes, coord_cartesian, element_blank, element_line, element_text, element_textbox_simple, factor, filter, font_add_google, geom_line, geom_point, geom_richtext, geom_textbox, ggplot, ggsave, if_else, labs, library, margin, mutate, panel.background = element_blank, paste0, scale_color_manual, scale_x_continuous, showtext_auto, showtext_opts, theme, tribble, and unit. The newsletter describing this visualization at a 30,000 ft view can be [found here](https://shop.riffomonas.org/posts/have-i-mentioned-that-i-love-slope-plots-here-s-one-from-the-nyt). You can find the original article presenting the [figure here](https://messaging-custom-newsletters.nytimes.com/dynamic/render?isViewInBrowser=true&paid_regi=1&productCode=NN&sendId=206871&uri=nyt%3A%2F%2Fnewsletter%2Ff4208c31-b5ec-5993-8455-1f449eeae86e). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

```R
library(tidyverse)
library(ggtext)
library(showtext)

font_add_google("Domine", "domine")
font_add_google("Libre Franklin", "franklin")
showtext_opts(dpi = 300)
showtext_auto()

d <- tribble(
  ~date, ~sentiment, ~percentage, ~label_nudge,
  2023, "enough", 30, 1.5,
  2023, "intentionally", 22, 3.5,
  2023, "unintentionally", 21, -2.8,
  2025, "enough", 25, -2.8,
  2025, "intentionally", 40, -4,
  2025, "unintentionally", 16, -2.8
) %>%
  mutate(
    label_text = if_else(
      sentiment == "enough",
      paste0(percentage, "%"),
      paste0("**", percentage, "%**")
    ),
    label_hjust = if_else(date == 2023, 0.15, 0.75)
  )

pretty_sentiment <- d %>%
  filter(date == 2025) %>%
  mutate(label = factor(
    sentiment,
    levels = c("enough", "intentionally", "unintentionally"),
    labels = c(
      "Israel is taking enough precautions to avoid civilian casualties",
      "**Intentionally**",
      "**Unintentionally**")
  ))


p <- d %>%
  ggplot(aes(x = date, y = percentage, group = sentiment, color = sentiment)) +
  geom_line() +
  geom_point() +
  geom_richtext(
    aes(label = label_text, hjust = label_hjust,),
    nudge_y = d$label_nudge, family = "franklin",
    label.color = NA,
    label.padding = margin(0, 0, 0, 0),
    fill = NA, size = 5
  ) +
  geom_textbox(
    data = pretty_sentiment,
    aes(label = label),
    nudge_y = pretty_sentiment$label_nudge, family = "franklin", size = 5,
    hjust = 0, lineheight = 0.9, 
    x = 2025.1,
    box.color = NA,
    box.padding = margin(0, 0, 0, 0),
    fill = NA,
    width = unit(140, "pt"),
  ) +
  labs(
    x = NULL, y = NULL,
    tag = "THE NEW YORK TIMES/SIENA POLL",
    title = "Dec. 2023 vs. Sept. 2025",
    subtitle = "Do you think Israel is intentionally or unintentionally killing civilians?",
    caption = "Based on The New York Times/Siena polls of registered voters nationwide conducted Dec. 10-14, 2023, and Sept. 22-27, 2025. | Respondents were asked if they thought Israel was taking enough precautions to avoid civilian casualties in Gaza. Those who said Israel was not taking enough precautions were asked if they thought Israel was intentionally or unintentionally killing civilians. | by Yuhan Liu"
  ) +
  scale_color_manual(
    breaks = c("intentionally", "unintentionally", "enough"),
    values = c("#BF4126", "#CC6704", "#A8A8A8")
  ) +
  scale_x_continuous(
    breaks = c(2023, 2025),
    labels = c("Dec. 2023", "Sept. 2025"),
  ) +
  coord_cartesian(
    xlim = c(2023, 2025),
    ylim = c(6, 40),
    clip = "off", expand = FALSE
  ) +
  theme(
    text = element_text(family = "franklin"),
    legend.position = "none",

    axis.text.y = element_blank(),
    axis.ticks.y = element_blank(),
    axis.text.x = element_text(
      hjust = c(0.15, 0.85), size = 12.5, color = "gray60",
      margin = margin(t = 8)
    ),
    axis.ticks.x = element_line(color = "gray60", linewidth = 0.3),
    axis.ticks.length.x = unit(7, "pt"),
    axis.line.x = element_line(color = "gray60", linewidth = 0.3),

    plot.margin = margin(t = 15, r = 180, b = 20, l = 40),
    plot.tag.location = "plot",
    plot.tag = element_text(
      family = "domine", face = "bold", size = 8.5, margin = margin(l = -20)
    ),
    plot.title.position = "plot",
    plot.title = element_text(
      margin = margin(t = 17, l = -20), size = 12, color = "gray60"
    ),

    plot.subtitle = element_textbox_simple(
      face = "bold", size = 15,
      margin = margin(t = 25, b = 25, r = -100, l = -20)),

    plot.caption = element_textbox_simple(
      family = "domine", color = "gray40",
      halign = 1, face = "bold", size = 8,
      margin = margin(r = -160, t = 30, l = -20)
    ),
    panel.background = element_blank(),
    panel.grid = element_blank()
  )

ggsave("israel_sentiment.png", width = 6, height = 5.945)
```