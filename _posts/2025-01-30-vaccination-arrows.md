---
layout: post
title: "Visualizing the effect of the pandemic on kindergarten vaccination rates with dplyr and ggplot2 (CC337)"
blurb: "The pandemic resulted in decreased vaccination rates"
author: "PD Schloss"
date: 2025-01-30 8:00
comments: false
youtube: 0hzWaIapoVo
start_hash: 
end_hash: 
---

Pat recreates a figure from the New York Times's UpShot showing the change US kindergarten vaccination rates because of the pandemic by state using tools from dplyr and ggplot2. The functions he uses from these packages include aes, annotate, arrange, as.numeric, coord_cartesian, drop_na, element_blank, element_line, element_rect, element_text, factor, filter, font_add_google, geom_segment, geom_text, ggplot, ggsave, if_else, labs, margin, mean, mutate, na_if, pivot_wider, pull, read_csv, scale_color_manual, scale_x_continuous, select, showtext_auto, showtext_opts, summarize, and theme. The original NYT news article is [available here](https://www.nytimes.com/interactive/2025/01/13/upshot/vaccination-rates.html). Pat's newsletter describing how he would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/data-visualization-as-a-vaccination-against-ignorance). The csv file can be [downloaded from the CDC](https://data.cdc.gov/Vaccinations/Vaccination-Coverage-and-Exemptions-among-Kinderga/ijqb-a7ye/about_data).

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)
library(showtext)

font_add_google("Libre Franklin", "franklin")
showtext_auto()
showtext_opts(dpi = 300)

#https://data.cdc.gov/Vaccinations/Vaccination-Coverage-and-Exemptions-among-Kinderga/ijqb-a7ye/about_data

state_cdc_data <- read_csv("Vaccination_Coverage_and_Exemptions_among_Kindergartners_20250118.csv") %>%
  filter(Geography %in% c(state.name, "United States") &
           `Vaccine/Exemption` == "MMR") %>%
  select(state = Geography,
         school_year = "School Year",
         estimate = "Estimate (%)") %>%
  filter(school_year %in% c("2017-18", "2018-19", "2019-20", "2023-24")) %>%
  mutate(estimate = na_if(estimate, "NR"),
         estimate = as.numeric(estimate),
         period = if_else(school_year != "2023-24",
                          "Prepandemic", school_year)) %>%
  summarize(mean_estimate = mean(estimate, na.rm = TRUE),
            .by = c(state, period)) %>%
  pivot_wider(names_from = "period", values_from = "mean_estimate") %>% 
  drop_na() %>%
  mutate(state = if_else(state == "New York", "New York  \u00B7", state),
         state = if_else(state == "Rhode Island", " Rhode Island" , state))

state_order <- state_cdc_data %>%
  filter(state != "United States") %>%
  arrange(-`2023-24`) %>%
  pull(state)

state_cdc_data %>%
  filter(state == "United States") 


state_cdc_data %>%
  mutate(state = factor(state,
                        levels = c(state_order, "United States"),
                        labels = c(state_order, "U.S. average")),
         yindex = as.numeric(state),
         left_right = if_else(Prepandemic < `2023-24`, "right", "left"),
         hjust = if_else(left_right == "left", 0, 1),
         x = if_else(Prepandemic < `2023-24`,
                     Prepandemic - 0.25, Prepandemic + 0.3),
         face = if_else(state == "U.S. average", "bold", "plain")) %>%
  ggplot(aes(x = Prepandemic, xend = `2023-24`, y = yindex,
             color = left_right)) +
  annotate(geom = "rect",
           fill = "#FFFFE8",
           xmin = 92.7, xmax = 95.0, ymin = 0.5, ymax = 51.5) +
  annotate(geom = "segment",
           x = c(92.7, 95.0), xend = c(92.7, 95.0),
           y = c(50, 50), yend =c(54, 54)) +
  annotate(geom = "text",
           hjust = c(0.9, 0.1),
           vjust = -0.3,
           x = c(92.7, 95), y = c(54, 54),
           family = "franklin", size = 9, size.unit = "pt",
           label = c("2023-2024", "PREPANDEMIC")) +
  geom_text(aes(x = x, label = state, hjust = hjust, fontface = face),
            size = 9, size.unit = "pt", color = "black", family = "franklin",
            show.legend = FALSE) +
  geom_segment(arrow = arrow(length = unit(5, "pt"), type = "closed"),
               linewidth = 1,
               show.legend = FALSE) +
  scale_color_manual(breaks = c("left", "right"),
                     values = c("#A86B5E", "#B5C1A8")) +
  scale_x_continuous(position = "top",
                     limits = c(79.5, 103),
                     breaks = seq(80, 100, 5),
                     labels = c("80%", "85%", "90%", "", "100%")) +
  coord_cartesian(ylim = c(0.5, 51.5), expand = FALSE, clip = "off") +
  labs(title = "Change in kindergarten measles vaccination rates") +
  theme(
    text = element_text(family = "franklin"),
    axis.text.y = element_blank(),
    axis.text.x = element_text(size = 10, color = "gray40"),
    axis.ticks = element_blank(),
    axis.title = element_blank(),
    panel.background = element_rect(fill = "white", color = "white"),
    panel.grid.minor = element_blank(),
    panel.grid.major.y = element_blank(),
    panel.grid.major.x = element_line(color = "gray80", linewidth = 0.25),
    plot.title = element_text(face = "bold", size = 14,
                              margin = margin(b = 30))
  )


ggsave("vaccination_arrows.png", width = 5, height = 7) 
```
