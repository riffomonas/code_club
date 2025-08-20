---
layout: post
title: "Modifying a published figure showing antibiotic discovery and resistance in R with ggplot2 (CC366)"
blurb: "An interesting gantt chart"
author: "PD Schloss"
date: 2025-08-20 9:00
comments: false
youtube: JVpRkyi9pN4
start_hash: 
end_hash: 
---

After seeing the same poorly laid out figure that looks like a gannt chart in a few seminars I decided to modify it to suit my style. After getting the original data from the published paper I recreate the original and customize it using R, ggplot2, and other tools from the tidyverse. The plot was generated using functions from the tidyverse including the dplyr, ggplot2, and rvest packages. The functions he uses from these packages include aes, annotate, as.numeric, coord_cartesian, drop_na, dup_axis, element_blank, element_line, element_text, geom_rect, geom_text, ggplot, ggsave, html_table, if_else, library, margin, mutate, n, nest, pivot_longer, pivot_wider, pluck, read_html, scale_fill_manual, scale_x_continuous, select, separate, seq, str_replace, theme, unit, and unnest. The newsletter describing this visualization at a 30,000 ft view can be found [here](https://shop.riffomonas.org/posts/visualizing-the-timeline-of-antibiotic-discovery-and-resistance-with-ggplot2). You can find the original article describing the data in the journal [Antibiotics](https://www.mdpi.com/2079-6382/11/9/1237). The data are availble in Table 1 of that paper. If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!


Want more practice on the concepts covered in Code Club? You can sign up for my weekly newsletter at https://shop.riffomonas.org/youtube to get practice problems, tips, and insights. If you're interested in purchasing a video workshop be sure to check out https://riffomonas.org/workshops/

Support Riffomonas by becoming a Patreon member!
https://www.patreon.com/riffomonas

You can also find complete tutorials for learning R with the tidyverse using...
Microbial ecology data: https://www.riffomonas.org/minimalR/
General data: https://www.riffomonas.org/generalR/

If you want to cite this video, please consider citing https://journals.asm.org/doi/10.1128/mra.01310-22



```R
library(tidyverse)
library(rvest)

url <- "https://www.mdpi.com/2079-6382/11/9/1237"
nudge <- 1

read_html(url) %>%
  html_table() %>%
  pluck(1) %>%
  select(class = `Antibiotic Class`,
         discovery_start = `Discovery Date`,
         clinical_start = `Clinical Use Date`,
         clinical_end = `Resistance Date`) %>%
  mutate(discovery_end = clinical_start) %>%
  pivot_longer(-class, names_to = "period", values_to = "year") %>%
  mutate(year = str_remove(year, " \\[.+\\]"),
         year = str_replace(year, "N/A 1", "2020"),
         year = as.numeric(year)) %>%
  separate(period, into = c("period", "position")) %>%
  pivot_wider(names_from = "position", values_from = "year") %>%
  nest(data = -class) %>%
  mutate(ymin = (n()-1):0,
         ymax = ymin + 1) %>%
  unnest(data) %>%
  mutate(xlabel = if_else(start[1] > 1910, start[1] - nudge, end[2] + nudge),
         hjust = if_else(start[1] > 1910, 1, 0), .by = class) %>%
  drop_na() %>%
  ggplot(aes(xmin = start, xmax = end,
             ymin = ymin, ymax = ymax, fill = period)) +
  geom_rect(color = "black", linewidth = 0.3) +
  geom_text(aes(x = xlabel, label = class, y = ymin + 0.5, hjust = hjust),
            size = 8, size.unit = "pt") +
  annotate(geom = "segment",
           x = seq(1900, 2020, 10),
           y = -0.7, yend = -0.3,
           linewidth = 0.3) +
  annotate(geom = "segment",
           x = seq(1900, 2020, 10),
           y = 38.7, yend = 38.3,
           linewidth = 0.3) +
  scale_fill_manual(
    name = NULL,
    breaks = c("discovery", "clinical"),
    values = c("#FFB380", "#86CDDE"),
    labels = c("Discovery window", "Clinical window")
  ) +
  coord_cartesian(
    xlim = c(1900, 2020),
    ylim = c(-0.5, 38.5), expand = FALSE, clip = "off") +
  scale_x_continuous(
    breaks = seq(1900, 2020, 10),
    sec.axis = dup_axis()) +
  theme(
    legend.position = "inside",
    legend.position.inside = c(0.15, 0.1),
    axis.title = element_blank(),
    axis.text.y = element_blank(),
    axis.text.x = element_text(color = "black"),
    axis.ticks = element_blank(),
    axis.line.x = element_line(linewidth = 0.3),
    panel.grid = element_blank(),
    panel.background = element_blank(),
    plot.margin = margin(t = 5, r = 15, b = 5, l = 10),
    legend.key.size = unit(13, "pt")
  )

ggsave("antibiotics.png", width = 6, height = 7.05)
```