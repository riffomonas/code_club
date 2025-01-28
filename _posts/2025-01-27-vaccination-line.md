---
layout: post
title: "Visualizing changes in kindergarten vaccination rates with dplyr and ggplot2 (CC336)"
blurb: "The pandemic resulted in decreased vaccination rates"
author: "PD Schloss"
date: 2025-01-27 8:00
comments: false
youtube: RCo85v7zt08
start_hash: 
end_hash: 
---

Pat recreates a figure from the New York Times's UpShot showing the change US kindergarten vaccination rates over the past 13 years using tools from dplyr and ggplot2. The functions he uses from these packages include annotate, coord_cartesian, element_text, filter, geom_hline, geom_line, geom_point, geom_text, ggsave, labs, margin, mutate, read_csv, scale_color_manual, scale_fill_manual, scale_x_continuous, scale_y_continuous, select, theme, theme_classic, tribble, and unit. The original NYT news article is [available here](https://www.nytimes.com/interactive/2025/01/13/upshot/vaccination-rates.html). Pat's newsletter describing how he would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/data-visualization-as-a-vaccination-against-ignorance). The csv file can be [downloaded from the CDC](https://data.cdc.gov/Vaccinations/Vaccination-Coverage-and-Exemptions-among-Kinderga/ijqb-a7ye/about_data).

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)
library(showtext)

font_add_google("Libre Franklin", "franklin")
showtext_auto()
showtext_opts(dpi = 300)

vaccines <- c("DTP, DTaP, or DT", "MMR", "Polio")


cdc_data <- read_csv("Vaccination_Coverage_and_Exemptions_among_Kindergartners_20250118.csv") %>%
  filter(Geography == "United States") %>%
  select(vaccine = "Vaccine/Exemption",
         school_year = "School Year",
         estimate = "Estimate (%)") %>%
  filter(vaccine %in% vaccines & 
           school_year != "2009-10" & school_year != "2010-11") %>%
  mutate(estimate = as.numeric(estimate),
         year = as.numeric(str_replace(school_year, "-..", "")))

cdc_label <- tribble(~label,~year,~estimate,~vaccine,
                     "Whooping\ncough", 2023.25, 92.15,"DTP, DTaP, or DT",
                     "Measles", 2023.25, 93.1,"MMR",
                     "Polio", 2023.25, 92.6, "Polio")

cdc_data %>%
  ggplot(aes(x = year, y = estimate,
             color = vaccine, fill = vaccine, group = vaccine)) +
  geom_hline(yintercept = 95, color = "black") + 
  geom_line(linewidth = 1, show.legend = FALSE) +
  annotate(geom = "segment",
           x = c(2023.2, 2023), xend = c(2023, 2023),
           y = c(93.1, 93.1), yend = c(93.1, 92.7)) +
  
  geom_point(size = 3, shape = 21, color = "white", show.legend = FALSE) +
  annotate(geom = "text", hjust = 1, vjust = -0.3,
           x = 2023.25, y = 95, label = "FEDERAL\nMEASLES TARGET",
           family = "franklin",
           lineheight = 0.8) +
  geom_text(data = cdc_label,
            mapping = aes(x = year, y = estimate,
                          color = vaccine, label = label),
            hjust = 0, lineheight = 0.8, size = 13, size.unit = "pt",
            family = "franklin", fontface = "bold",
            show.legend = FALSE) +
  scale_y_continuous(limits = c(90.9, 95.5),
                     breaks = 91:95,
                     labels = c(91:94, "95%")) +
  scale_x_continuous(breaks = c(2011, 2015, 2019, 2023),
                     labels = c("2011-12", "2015-16", "2019-20", "2023-24")) +
  coord_cartesian(xlim = c(2009.5, 2023.25),
                  expand = FALSE, clip = "off") +
  scale_color_manual(breaks = vaccines,
                     values = c("#F1C40F", "#B5C1A8", "#A86B5E")) +
  scale_fill_manual(breaks = vaccines,
                     values = c("#F1C40F", "#B5C1A8", "#A86B5E")) +
  labs(x = "SCHOOL YEAR",
       y = NULL,
       title = "Share of U.S. kindergartners vaccinated against ...",
       caption = "Source: Centers for Disease Control and Prevention") +
  theme_classic() +
  theme(
    plot.margin = margin(t = 8, r = 70, b = 3, l = 8),
    plot.title = element_text(face = "bold", size = 16,
                              margin = margin(b = 25)),
    plot.caption = element_text(face = "bold", color = "gray60", hjust = 0),
    text = element_text(family = "franklin"),
    panel.grid.major.y = element_line(),
    axis.line = element_blank(),
    axis.ticks.y = element_blank(),
    axis.ticks.length.x = unit(5, "pt"),
    axis.ticks.x = element_line(linewidth = 0.25),
    axis.text.y = element_text(vjust = -0.3, hjust = 0, size = 12.5,
                               margin = margin(r = -30)),
    axis.text.x = element_text(size = 12.5,
                               margin = margin(t = 5, b = 10))
  )

ggsave("vaccination_line.png", width = 6, height = 4.6875) 
```
