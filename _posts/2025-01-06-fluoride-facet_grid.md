---
layout: post
title: "Creating side-by-side line plots with ggplot2's facet_grid function in R (CC330)"
blurb: "Facted slope plots"
author: "PD Schloss"
date: 2025-01-06 8:00
comments: false
youtube: 3L_Cvx6rIFA
start_hash: 
end_hash: 
---

Pat recreates a visualization from CNN showing the change in dental problems over time in countries that vary in whether they fluoridate their water. To pull off this visualization, Pat uses the ggplot2 and ggtext packages and relies heavily on geom_line, geom_point, geom_text, and facet_grid. In addition to these functions, he also uses the labs, scale_y_continuous, scale_x_continuous, scale_color_manual, coord_cartesian, theme_classic, theme functions. The original article can be found on the [CNN website](https://www.cnn.com/2024/11/23/health/fluoride-drinking-water-dg/index.html) and the CAPP database can be [found here](https://mau.se/en/about-us/faculties-and-departments/faculty-of-odontology/oral-health-countryarea-profile-project--capp/). Pat's newsletter describing how he would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/visualizing-the-effects-of-fluoride-in-drinking-water).

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)
library(ggtext)

dmft_data <- tribble(
  ~country, ~year, ~dmft, ~fluoridates_water,
  "Brazil",1976,9.89,TRUE,
  "Brazil",2006,0.9,TRUE,
  "New\nZealand",1977,7,TRUE,
  "New\nZealand",2022,0.72,TRUE,
  "Ireland",1984,3.3,TRUE,
  "Ireland",2002,1.4,TRUE,
  "USA",1980,2.6,TRUE,
  "USA",2004,1.2,TRUE,
  "Iceland",1982,8.3,FALSE,
  "Iceland",2005,1.4,FALSE,
  "Italy",1979,6.9,FALSE,
  "Italy",2005,1.1,FALSE,
  "Japan",1975,5.9,FALSE,
  "Japan",2021,0.63,FALSE,
  "UK",1983,3.1,FALSE,
  "UK",2013,0.8,FALSE
) %>% 
  mutate(fluoridates_water = factor(fluoridates_water, levels = c(TRUE, FALSE)))

dmft_labels <- dmft_data %>%
  slice_min(year, n = 1, by = country)

fluoridates_labels <- c(
  "TRUE" = "<span style='color:navy'>Fluoridates water</span>",
  "FALSE" = "<span style='color:dodgerblue'>Does not fluoridate water</span>"
)

dmft_data %>%
  ggplot(aes(x = year, y = dmft, color = fluoridates_water,
             label = country, group = country)) +
  geom_line(linewidth = 0.3) +
  geom_point(size = 0.7) +
  geom_text(data = dmft_labels, mapping = aes(x = year - 3.5),
            color = "gray50", size = 7, size.unit = "pt",
            lineheight = 0.8) +
  facet_grid(~fluoridates_water, axes = "all_y",
             labeller = labeller(fluoridates_water = fluoridates_labels)) +
  labs(
    title = "Decayed, missing, and filled teeth at age 12, by country",
    caption = "Note: Data is based on national oral health surveys, national health reports, national registers, personal communication and scientific journals.<br><br>
Source: Malmö University's Oral Health Country/Area Profile Project (CAPP)<br>
Graphic: Pat Schloss (after Soph Warnes, CNN)",
    x = NULL,
    y = NULL
  ) +
  scale_y_continuous(limit = c(0, 10),
                     breaks = seq(2, 10, 2)) +
  scale_x_continuous(limit = c(1970, 2023),
                     breaks = seq(1970, 2030, 10)) +
  scale_color_manual(breaks = c(FALSE, TRUE),
                     values = c("dodgerblue", "navy")) +
  coord_cartesian(expand = FALSE, clip = "off") +
  theme_classic() +
  theme(legend.position = "none",
        plot.title.position = "plot",
        plot.title = element_text(size = 9, face = "bold"),
        plot.caption.position = "plot",
        plot.caption = element_textbox_simple(hjust = 0, color = "darkgray",
                                              size = 7, margin = margin(t = 21)),
        axis.line.y = element_blank(),
        axis.ticks.y = element_blank(),
        axis.text = element_text(size = 6.5, color = "darkgray"),
        axis.ticks.x = element_line(color = "darkgray", linewidth = 0.1),
        axis.ticks.length.x = unit(4, "pt"),
        axis.line.x = element_line(color = "darkgray", linewidth = 0.4), 
        panel.grid.major.y = element_line(color = "darkgray", linewidth = 0.1),
        strip.background = element_blank(),
        strip.text = element_markdown(hjust = 0, face = "bold", size = 7,
                                      margin = margin(b = 8, l = -13)),
        strip.clip = "off"
        )

ggsave("fluoride.png", width = 6, height = 3)
```
