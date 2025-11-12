---
layout: post
title: "Creating a stacked bar plot infographic from Our World in Data in R with ggplot2 (CC381)"
blurb: "If you insist on making a stacked bar plot"
author: "PD Schloss"
date: 2025-11-12 12:00
comments: false
youtube: 9jWuC_GciLI
start_hash: 
end_hash: 
---

Pat recreates a stacked barplot that has an inforgraphic feel to it from Our World in Data. Aside from the stacked bar plot, he also shows how to label each rectangle, change the size of the font, and do some wacky stuff with facets and arrows. He recreated the figure using R, ggplot2, showtext, ggtext, and other tools from the tidyverse. The functions he used from these packages included aes, annotate, annotation_custom, arrow, coord_cartesian, download.file, element_blank, element_markdown, element_text, element_textbox_simple, facet_wrap, factor, fct_reorder, font_add_google, function, geom_col, geom_text, ggplot, ggsave, glue, if_else, labs, library, margin, mutate, pivot_longer, position_stack, rasterGrob, readPNG, read_csv, rsvg_png, scale_fill_manual, scale_size_identity, scale_x_discrete, select, showtext_auto, showtext_opts, str_detect, theme, unit, and unzip. The newsletter describing this visualization at a 30,000 ft view can be [found here](https://shop.riffomonas.org/posts/visualizing-how-you-re-most-likely-to-die-vs-what-the-media-wants-you-to-think). You can find the original article presenting the figure at [the Our World in Data website](https://ourworldindata.org/does-the-news-reflect-what-we-die-from). Here's a video critiquing the original plot https://youtu.be/VCq53oc7JQQ. If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```R
library(tidyverse)
library(glue)
library(showtext)
library(ggtext)
library(rsvg)

font_add_google("Lato", "lato")
font_add_google("Playfair Display", "playfair", bold.wt = 600)
showtext_opts(dpi = 300)
showtext_auto()


logo_svg <- "https://ourworldindata.org/owid-logo.svg"
rsvg_png(logo_svg, "owid_logo.png", width = 2000)

get_png <- function(filename) {
  grid::rasterGrob(png::readPNG(filename), interpolate = TRUE)
}

l <- get_png("owid_logo.png")



data_url <- "https://catalog.owid.io/analyses/media-deaths-analysis-data.zip"
data_zip <- "media-deaths-analysis-data.zip"
download.file(data_url, data_zip)

unzip(data_zip)

read_csv("data/media_deaths_results.csv") %>%
  select(
    cause,
    actual = deaths_share,
    fox = fox_share,
    nyt = nyt_share,
    wapo = wapo_share) %>%
  mutate(
    full_cause = c("Accidents", "Alzheimer's disease", "Cancer", "COVID-19",
                   "Diabetes", "Drug overdose", "Heart disease", "Homicide",
                   "Influenza/Pneumonia", "Kidney failure",
                   "Liver disease", "Lower respiratory diseases\n", "Stroke",
                   "Suicide", "Terrorism"),
    cause = fct_reorder(cause, -actual)) %>%
  pivot_longer(-c(cause, full_cause), names_to = "source", values_to = "share") %>%
  mutate(
    type = if_else(
      source == "actual",
      "",
      "Media coverage of these causes of death in 2023 in..."),
    pretty_label = if_else(
      (str_detect(type, "Media") & share < 2.2) | (type == "" & share < 1),
      "",
      glue("{full_cause} ({signif(share, 2)}%)")
      ),
    label_size = if_else(share < 3, 4, 5.5),
    source = factor(source, levels = c("actual", "nyt", "wapo", "fox"))
  ) %>%
  ggplot(
    aes(x = source, y = share, fill = cause, label = pretty_label,
        size = label_size)) +
  geom_col(show.legend = FALSE, width = 0.8) +
  geom_text(
    color = "white", family = "lato",
    size.unit = "pt", lineheight = 0.9,
    position = position_stack(vjust = 0.5)) +
  facet_wrap(~type, scale = "free_x", space = "free_x") +
  scale_size_identity() +
  scale_fill_manual(
    breaks = c(
      "heart disease", "cancer", "accidents", "stroke", "respiratory",  
      "alzheimers", "diabetes", "kidney", "liver", "covid", "suicide",
      "influenza", "drug overdose", "homicide", "terrorism"),
    values = c(
      "#7188B0", "#9F5961", "#799A6A", "#C15D39", "#C18143",
      "#AC336B", "#DF6373", "#33547C", "#339D98", "#C9A57A", "#7EA189",
      "#BE8E95", "#B577B0", "#79BDA3", "#456C3F"
    )
  ) +
  scale_x_discrete(
    position = "top",
    breaks = c("actual", "nyt", "wapo", "fox"),
    labels = c(
      "<span style='color:#6D3E91'>**Causes of death**<br>in the US in 2023</span>",
      "<span style='color:#B13508'>The New York Times</span>",
      "<span style='color:#B13508'>The Washington Post</span>",
      "<span style='color:#B13508'>Fox News</span>")) +
  labs(
    title = "What Americans die from",
    subtitle = "and the causes of death the US media reports on",
    caption = "<p><b>Note:</b> Based on the share of causes of death in the US and the share of mentions for each of the causes in the New York Times, the Washington Post, and Fox News. All values are normalized to 100%, so the shares are relative to all deaths caused by the 12 most common causes + drug overdoses, homicides, and terrorism. These causes account for more than 75% of deaths in the US.<br>
A \"media mention\" is a published article in one of the outlets which mentions the cause (e.g., \"influenza\") or related keywords (e.g., \"flu\") at least twice.</p>
<p><b>Data sources:</b> Media mentions from Media Cloud (2025); deaths data from the US CDC (2025) and Global Terrorism Index.</p>",
    tag = "CC BY",
    x = NULL,
    y = NULL
  ) +
  coord_cartesian(
    expand = FALSE, clip = "off",
    xlim = c(0.55, NA), ylim = c(0, 100)) +
  annotate(
    geom = "text", size = 3.5, size.unit = "pt",
    x = 0.58, y = c(2, 0), label = c("Homicide (<1%)", "Terrorism (<0.001%)"),
    color = c("#79BDA3", "#456C3F"), hjust = 1, layout = 1
  ) +
  annotate(
    geom = "curve",
    arrow = arrow(type = "closed", angle = 20, length = unit(10, "pt")),
    color = "#6D3E91", x = 0.53, xend = 0.6, y = 125, yend = 105, layout = 1
  ) +
  annotate(
    geom = "curve",
    arrow = arrow(type = "closed", angle = 20, length = unit(10, "pt")),
    color = "#8B2A09", x = 3.5, xend = 3.45, y = 115, yend = 97, layout = 2,
    curvature = -0.5
  ) +
  theme(
    text = element_text(family = "lato"),
    plot.title = element_text(
      family = "playfair", size = 15, color = "#6D3E91", face = "bold",
      margin = margin(b = 3)
    ),
    plot.subtitle = element_text(
      family = "playfair", size = 15, hjust = 1, color = "#B13508",
      face = "bold"
    ),
    plot.caption = element_textbox_simple(
      color = "gray60", size = 4.75,
      margin = margin(l = -35, r = -10, t = 10)),
    plot.caption.position = "plot",
    plot.tag = element_text(color = "gray60", size = 5),
    plot.tag.location = "plot",
    plot.tag.position = "bottomright",
    
    plot.margin = margin(t = 7, r = 34, b = 3, l = 42),
    
    strip.placement = "outside",
    strip.text = element_text(
      face = "bold", color = "#B13508", margin = margin(t = 0, b = -5)
    ),
    strip.background = element_blank(),
    strip.clip = "off",
    
    axis.text.x.top = element_markdown(size = 7),
    axis.ticks = element_blank(),
    axis.text.y = element_blank(),
    panel.spacing.x = unit(45, "pt"),
    panel.grid = element_blank(),
    panel.background = element_blank()
  ) +
  annotation_custom(
    l,
    xmin = 4.8, xmax = 5.2,
    ymax = 250
  )

ggsave("media_deaths.png", width = 6, height = 4.235)
```