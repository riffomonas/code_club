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

data <- read_csv(list.files(pattern = "csv"), id = "group",
         col_names = c("year", "israelis", "palestinians"),
         skip = 1) %>%
  mutate(group = str_replace(group, "\\'.*", ""),
         group = if_else(group == "Americans", "All\nAmericans", group),
         israelis = str_replace(israelis, "%", "") %>% as.numeric,
         palestinians = str_replace(palestinians, "%", "") %>% as.numeric,
         pro_israelis = israelis - palestinians,
         date = ymd(paste(year, "01-01", sep = "-"))
         )

data %>%
  ggplot(aes(x = date, y = pro_israelis, color = group)) +
  geom_hline(yintercept = c(-40, 0, 40, 80),
             linewidth = c(0.3, 0.3, 0.3, 0.3),
             color = c("gray80", "black", "gray80", "gray80")) +
  geom_line(linewidth = 1, 
            show.legend = FALSE) +
  annotate(geom = "segment", 
           x = ymd(c("2016-01-01", "2023-10-06")),
           y = c(-40, -40), yend = c(80, 80),
           color = "gray60", linewidth = 0.3) +
  annotate(geom = "text",
           x = ymd(c("2016-01-01", "2023-10-07")), y = -25, 
           label = c("2016", "Oct. 7, 2023"), 
           hjust = 1.1, size = 9, size.unit = "pt", family = "franklin") +
  geom_richtext(data = tibble(x = ymd("2000-02-01"), y = c(4, -4),
                              label = c("More support for **Israelis**",
                                        "Less support for **Palestinians**")),
                aes(x = x, y = y, label = label), inherit.aes = FALSE,
                hjust = 0, label.size = 0, fill = NA, family = "franklin",
                size = 3) +
  geom_text(data = filter(data, year == 2025),
            aes(y = pro_israelis + 0.75, label = group),
            x = ymd("2025-10-01"),
            hjust = 0, vjust = 1, 
            family = "franklin", size = 9, size.unit = "pt",
            show.legend = FALSE) +
  geom_segment(data = filter(data, year == 2025),
               aes(y = pro_israelis),
               x = ymd("2025-01-01"), xend = ymd("2025-06-01"),
               show.legend = FALSE, linetype = "11") +
  scale_color_manual(breaks = c("Republicans", "All\nAmericans", "Democrats"),
                     values = c("#BE2C25", "#494949", "#1366B3")) +
  scale_y_continuous(breaks = c(-40, 0, 40, 80),
                     limits = c(-40, NA),
                     labels = c("-40", "\u00B10", "+40", "+80")) +
  scale_x_date( breaks = seq(ymd("2005-01-01"), ymd("2025-01-01"), "5 years"),
                limits = ymd(c("2000-01-01", "2025-10-01")),
                date_labels = c("2005", "2010", "2015", "2020", "2025")) +
  coord_cartesian(clip = "off", expand = FALSE) +
  labs(x = NULL, y = NULL,
       title = "Sympathy for Palestinians has surged since 2016 -- driven by Democrats",
       subtitle = "How much more likely Americans and members of the two major parties were to express sympathy with Israelis over Palestinians.",
       caption = "Question text: \"In the Middle East situation, are your sympathies more with the Israelis or more with the Palestinians?\"<br><br><span style='font-size: 8pt;'>Source: Gallup, March 2025</span>"
  ) +
  theme(
    plot.margin = margin(t = 5, r = 52),
    text = element_text(family = "franklin"),
    plot.title.position = "plot",
    plot.title = element_textbox_simple(face = "bold", size = 13),
    plot.subtitle = element_textbox_simple(size = 11,
                                           margin = margin(t = 13, r = -48, b = 12)),
    plot.caption.position = "plot", 
    plot.caption = element_textbox_simple(size = 9.2, margin = margin(t = 13, r= -48)),
    panel.grid = element_blank(),
    panel.background = element_blank(),
    axis.ticks = element_blank(),
    axis.text.x = element_text(size = 9, margin = margin(t = 4), color = "black"),
    axis.text.y = element_text(size = 9, color = "black")
  )

ggsave("israelis-palestinians.png", width = 6, height = 6.97)
```
