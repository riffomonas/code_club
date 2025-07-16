---
layout: post
title: "Using ggplot2 to recreate a visual showing changes in US sentiment towards Israelis/Palestenians (CC363)"
blurb: "Making a line plot more sophisticated"
author: "PD Schloss"
date: 2025-07-16 9:00
comments: false
youtube: VXGKsuxchFg
start_hash: 
end_hash: 
---

Pat recreates a line plot from Philip Bump showing the change in sentiment among Americans towards Israelis and Palestinians over the past 25 years using the ggplot2 R package. The plot was generated using functions from the tidyverse including the dplyr, ggplot2, ggimage, showtext, ggtext, and glue packages. The functions he uses from these packages include aes, annotate, coord_cartesian, element_blank, element_text, element_textbox_simple, font_add_google, geom_hline, geom_line, geom_richtext, geom_segment, geom_text, ggplot, ggsave, if_else, labs, library, margin, mutate, paste, read_csv, scale_color_manual, scale_x_date, scale_y_continuous, showtext_auto, showtext_opts, str_replace, theme, tibble, and ymd.The newsletter describing this visualization at a 30,000 ft view can be found [here](https://shop.riffomonas.org/posts/making-a-basic-line-plot-appear-more-sophisticated). You can find the original article describing the data [here](https://s2.washingtonpost.com/camp-rw/?trackId=59f436769bbc0f5649e05510&s=685d32be2b51f26b00835603). The data are availble [here](https://news.gallup.com/poll/657404/less-half-sympathetic-toward-israelis.aspx). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

```R
library(tidyverse)
library(showtext)
library(ggtext)

font_add_google("Libre Franklin", "franklin")
showtext_opts(dpi = 300)
showtext_auto()

csv_files <- list.files(pattern = ".*Sympathies.*.csv")

gallup_data <- read_csv(csv_files, id = "party", skip = 1,
         col_names = c("year", "israelis", "palestinians")) %>%
  mutate(
    party = str_replace(party, "' Sympathies.*", ""),
    party = if_else(party == "Americans", "All\nAmericans", party),
    israelis = str_replace(israelis, "%", "") %>% as.numeric,
    palestinians = str_replace(palestinians, "%", "") %>% as.numeric,
    difference = israelis - palestinians,
    date = as.Date(paste0(year, "-01-01"))
  ) %>%
  select(party, year, date, difference)

gallup_data %>%
  ggplot(aes(x = date, y = difference, color = party)) +
  geom_text(data = filter(gallup_data, year == 2025),
            aes(label = party, y = difference + 1.5), x = as.Date("2025-10-01"),
            hjust = 0, vjust = 1,
            family = "franklin", size = 9, size.unit = "pt",
            lineheight = 0.8) +
  geom_segment(data = filter(gallup_data, year == 2025),
               aes(xend = as.Date("2025-08-01")),
               linetype = "21", linewidth = 0.4) +
  geom_hline(yintercept = seq(-40, 80, 40),
             linewidth = 0.25,
             color = c("gray80", "black", "gray80", "gray80")) +
  geom_line(linewidth = 1) +
  annotate(geom = "segment",
           x = as.Date(c("2016-01-01", "2023-10-07")),
           y = -40, yend = 80,
           linewidth = 0.25,
           color = "gray80") +
  annotate(geom = "text",
           x = as.Date(c("2016-01-01", "2023-10-07")),
           y = -25,
           label = c("2016", "Oct. 7, 2023"),
           family = "franklin", size = 8.5, size.unit = "pt",
           hjust = 1.1) +
  annotate(
    geom = "richtext",
    x = as.Date(c("2000-07-01", "2000-07-01")),
    y = c(-4, 4),
    label = c("More support for **Palestinians**",
              "More support for **Israelis**"),
    hjust = 0, family = "franklin", size = 3.25, label.size = 0
  ) +
  labs(
    title = "Sympathy for Palestinians has surged since 2016 -- driven by Democrats",
    subtitle = "How much more likely Americans and members of the two major parties were to express sympathy with Israelis over Palestinians.",
    caption = "Question text: \"In the Middle East situation, are your sympathies more with the Israelis or more with the Palestinians?\"<br><br><span style='font-size:8.5pt;'>Source: Gallup, March 2025</span>"
  ) +
  scale_y_continuous(
    limits = c(-40, NA),
    breaks = seq(-40, 80, 40),
    labels = c("-40", "\u00B10", "+40", "+80")
  ) +
  scale_x_date(
    limits = as.Date(c("2000-01-01", "2025-10-01")),
    breaks = as.Date(paste0(seq(2005, 2025, 5), "-01-01")),
    date_labels = "%Y"
  ) +
  coord_cartesian(expand = FALSE, clip = "off") +
  scale_color_manual(
    breaks = c("Republicans", "All\nAmericans", "Democrats"),
    values = c("#BC2B24", "#494949", "#1366B3")
  ) +
  theme(
    text = element_text(family = "franklin"),
    axis.ticks = element_blank(),
    axis.title = element_blank(),
    axis.text = element_text(color = "black", size = 9),
    panel.grid = element_blank(),
    panel.background = element_blank(),
    plot.title.position = "plot",
    plot.title = element_textbox_simple(face = "bold", size = 13.5,
                                        margin = margin(t = 5, r = -48)),
    plot.subtitle = element_textbox_simple(
      margin = margin(t = 23, b = 10, r= -48),
      size = 11.25, lineheight = 1.3),
    plot.caption.position = "plot",
    plot.caption = element_textbox_simple(size = 9,
                                          margin = margin(t = 13, r = -45)),
    plot.margin = margin(t = 5, r = 53, b = 0, l = 0),
    legend.position = "none"
  )

ggsave("israelis-palestinians.png", width = 6, height = 6.97)
```
