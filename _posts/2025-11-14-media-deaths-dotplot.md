---
layout: post
title: "Converting a stacked bar plot to a dot plot in R with ggplot2 (CC382)"
blurb: "Dot plots are superior to stacked bar plot"
author: "PD Schloss"
date: 2025-11-14 12:00
comments: false
youtube: ls14mnBCwI8
start_hash: 
end_hash: 
---

Pat converts a stacked bar plot into a dot plot and gives it an inforgraphic feel like we'd see on the Our World in Data website. Aside from the dot plot, he also shows how to annotate the figure with text and arrows, reorder factors by different variables, and embed colors in titles. He created the figure using R, ggplot2, showtext, ggtext, and other tools from the tidyverse. The functions he used from these packages included aes, annotate, arrow, as.numeric, coord_cartesian, download.file, element_blank, element_text, element_textbox_simple, factor, fct_reorder, filter, font_add_google, geom_curve, geom_point, ggplot, ggsave, library, margin, mutate, pivot_longer, position_dodge, read_csv, scale_color_manual, scale_x_continuous, scale_y_continuous, select, showtext_auto, showtext_opts, unit, and unzip. The newsletter describing this visualization at a 30,000 ft view can be [found here](https://shop.riffomonas.org/posts/visualizing-how-you-re-most-likely-to-die-vs-what-the-media-wants-you-to-think). You can find the original article presenting the figure on the [OWID site](https://ourworldindata.org/does-the-news-reflect-what-we-die-from). Here's a [video critiquing](https://youtu.be/VCq53oc7JQQ) the original plot. If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```R
library(tidyverse)
library(showtext)
library(ggtext)

font_add_google("Playfair Display", "playfair")
font_add_google("Lato", "lato")
showtext_opts(dpi = 300)
showtext_auto()

data_url <- "https://catalog.owid.io/analyses/media-deaths-analysis-data.zip"
data_zip <- "media-deaths-analysis-data.zip"

download.file(data_url, data_zip)
unzip(data_zip, exdir = "owid")

segment_bump <- 0.25

d <- read_csv("owid/data/media_deaths_results.csv") %>%
  select(deaths_share, fox_share, wapo_share, nyt_share) %>%
  mutate(
    ave_media = ((nyt_share + wapo_share + fox_share) / 3),
    pretty_cause = c("Accidents", "Alzheimer's disease", "Cancer", "COVID-19",
                     "Diabetes", "Drug overdose", "Heart disease", "Homicide",
                     "Influenza/Pneumonia", "Kidney failure", "Liver disease",
                     "Lower respiratory diseases", "Stroke", "Suicide",
                     "Terrorism"),
    pretty_cause = fct_reorder(pretty_cause, ave_media)
  ) %>%
  select(-ave_media) %>%
  pivot_longer(-pretty_cause, names_to = "source", values_to = "share") %>%
  mutate(
    source = factor(
      source, 
      levels = c("deaths_share", "fox_share", "wapo_share", "nyt_share")))

d %>%
  ggplot(aes(color = source, x = share, y = as.numeric(pretty_cause))) +
  # geom_segment(
  #   data = filter(d, source == "deaths_share"), color = "black",
  #   linewidth = 0.3,
  #   aes(
  #     y = as.numeric(pretty_cause) - segment_bump,
  #     yend = as.numeric(pretty_cause) + segment_bump)) +
  geom_point(
    data = filter(d, source == "deaths_share"), size = 3, color = "gray40") +
  geom_point(
    data = filter(d, source != "deaths_share"),# shape = "|", size = 3,
    position = position_dodge(width = 0.5, orientation = "y")) +
  labs(
    x = "Share of deaths (%)",
    y = NULL,
    title = "If it bleeds it leads...",
    subtitle = "In 2023, the coverage of violent deaths in
    <span style = 'color: #46ACC8;'>**The New York Times**</span>,
    <b style = 'color:#DD8D29;'>The Washington Post</b>, and 
    <span style = 'color:#FD6467'>**Fox News**</span> was disproportionate to
    the <span style='color: black'>**actual causes**</span> of death",
    caption = "<p><b>Note:</b> Based on the share of causes of death in the US
    and the share of mentions for each of the causes in the New York Times, the
    Washington Post, and Fox News. All values are normalized to 100%, so the
    shares are relative to all deaths caused by the 12 most common causes + 
    drug overdoses, homicides, and terrorism. These causes account for more than
    75% of deaths in the US.<br>A \"media mention\" is a published article in 
    one of the outlets which mentions the cause (e.g., \"influenza\") or related
    keywords (e.g., \"flu\") at least twice.</p>
    <p><b>Data sources:</b> Media
    mentions from Media Cloud (2025); deaths data from the US CDC (2025) and
    Global Terrorism Index.</p>"
  ) +
  scale_color_manual(
    breaks = c("nyt_share", "wapo_share", "fox_share"),
    values = c("#46ACC8", "#DD8D29", "#FD6467")
  ) +
  scale_x_continuous(position = "top") +
  scale_y_continuous(
    breaks = 1:15,
    labels = levels(d$pretty_cause)
  ) +
  coord_cartesian(
    expand = FALSE, clip = "off",
    xlim = c(-1, 57), ylim = c(0.5, 15.5)) +
  theme(
    text = element_text(family = "lato"),
    plot.title.position = "plot",
    plot.caption.position = "plot",
    
    plot.title = element_text(
      family = "playfair", face = "bold", size = 18, color = "red3"),
    plot.subtitle = element_textbox_simple(
      family = "playfair", size = 14, lineheight = 1.1, color = "gray40",
      margin = margin(t = 3, b = 15)),
    plot.caption = element_textbox_simple(
      color = "gray60", size = 6, margin = margin(t = 10)),
    
    legend.position = "none",
    
    panel.grid = element_blank(),
    panel.background = element_blank(),
    
    axis.ticks = element_blank(),
    axis.text.y = element_text(color = "black"),
    axis.text.x.top = element_text(color = "black", size = 8),
    axis.title = element_text(color = "black")
  ) +
  annotate(
    geom = "text",
    x = 38, y = 6, label = "Actual US share\nof deaths", hjust = 0,
    lineheight = 0.9, size = 8, size.unit = "pt", family = "lato"
  ) +
  geom_curve(
    x = 37, y = 6, xend = 29.75, yend = 7.6, color = "black",
    curvature = -0.4, linewidth = 0.3,
    arrow = arrow(type = "closed", length = unit(5, "pt"), angle = 20)) +
  annotate(
    geom = "text",
    x = 38, y = 13, label = "Media reported\nshare of deaths", hjust = 1,
    lineheight = 0.9, size = 8, size.unit = "pt", family = "lato"
  ) +
  geom_curve(
    x = 39, y = 13, xend = 46, yend = 14.5, color = "black",
    curvature = 0.5, linewidth = 0.3,
    arrow = arrow(type = "closed", length = unit(5, "pt"), angle = 20))

ggsave("media_deaths_dotplot.png", width = 5, height = 6)
```