---
layout: post
title: "Recreating a side-by-side line plot from CNN using patchwork and ggplot2 in R (CC331)"
blurb: "Patchworked slope plots"
author: "PD Schloss"
date: 2025-01-09 8:00
comments: false
youtube: xpxrX1x7y2g
start_hash: 
end_hash: 
---

Pat recreates a visualization from CNN showing the change in dental problems over time in countries that vary in whether they fluoridate their water. To pull off this visualization, Pat uses patchwork and the ggplot2 and ggtext packages. He relies heavily on geom_line, geom_point, geom_text, and plot_annotation. In addition to these functions, he also uses the labs, scale_y_continuous, scale_x_continuous, scale_color_manual, coord_cartesian, theme_classic, theme functions. Code for the episode that used facet_grid to generate the same figure can be [found here](https://riffomonas.org/code_club/2025-01-06-fluoride-facet_grid.md). The original CNN article can be [found here](https://www.cnn.com/2024/11/23/health/fluoride-drinking-water-dg/index.html) and the CAPP database can be [found here](https://mau.se/en/about-us/faculties-and-departments/faculty-of-odontology/oral-health-countryarea-profile-project--capp/). Pat's newsletter describing how he would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/visualizing-the-effects-of-fluoride-in-drinking-water).

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)
library(patchwork)
library(ggtext)

dmft_data <- tribble(
  ~country, ~year, ~dmft, ~fluoridates_water, ~label_x, ~label_y,
  "Brazil",1976,9.89,TRUE,1979.25,10,
  "Brazil",2006,0.9,TRUE,NA,NA,
  "New\nZealand",1977,7,TRUE,1975,8,
  "New\nZealand",2022,0.72,TRUE,NA,NA,
  "Ireland",1984,3.3,TRUE,1980.5,3.75,
  "Ireland",2002,1.4,TRUE,NA,NA,
  "USA",1980,2.6,TRUE,1977,2.6,
  "USA",2004,1.2,TRUE,NA,NA,
  "Iceland",1982,8.3,FALSE,1978,8.5,
  "Iceland",2005,1.4,FALSE,NA,NA,
  "Italy",1979,6.9,FALSE,1976,7.1,
  "Italy",2005,1.1,FALSE,NA,NA,
  "Japan",1975,5.9,FALSE,1974,5.4,
  "Japan",2021,0.63,FALSE,NA,NA,
  "UK",1983,3.1,FALSE,1981,3.2,
  "UK",2013,0.8,FALSE,NA,NA,
)

draw_slope_plot <- function(data, logical_fluoridates) {
  
  title_text <- if_else(logical_fluoridates,
                        "Fluoridates water",
                        "Does not fluoridate water")

  title_color <- if_else(logical_fluoridates,
                        "navy",
                        "dodgerblue")
  
  dmft_filtered <- data %>%
    filter(fluoridates_water == logical_fluoridates)
  
  dmft_label <- dmft_filtered %>%
    drop_na()

  dmft_filtered %>%
    ggplot(aes(x = year, y = dmft, color = fluoridates_water, group = country,
               label = country)) +
    geom_line(linewidth = 0.3) +
    geom_point(size = 0.7) +
    geom_text(data = dmft_label,
              mapping = aes(x = label_x, y = label_y),
              color = "gray50", size = 7, size.unit = "pt",
              lineheight = 0.9) +
    scale_color_manual(breaks = c(FALSE, TRUE),
                       values = c("dodgerblue", "navy")) +
    scale_y_continuous(breaks = seq(2, 10, 2),
                       limits = c(0, 10)) +
    scale_x_continuous(limits = c(1970, 2023),
                       breaks = seq(1970, 2030, 10)) +
    coord_cartesian(expand = FALSE, clip = "off") +
    labs(x = NULL, y = NULL,
         title = title_text) +
    theme_classic() +
    theme(
      legend.position = "none",
      plot.title.position = "plot",
      plot.title = element_text(color = title_color, face = "bold", size = 7,
                                margin = margin(t = 2, b = 8)),
      axis.line.x = element_line(linewidth = 0.4, color = "darkgray"),
      axis.line.y = element_blank(),
      axis.ticks.y = element_blank(),
      axis.ticks.x = element_line(linewidth = 0.2, color = "darkgray"),
      axis.ticks.length.x = unit(4, "pt"),
      panel.grid.major.y = element_line(linewidth = 0.1, color = "darkgray"),
      axis.text = element_text(size = 6.5, color = "darkgray")
    )

}

fluoridates <- draw_slope_plot(dmft_data, TRUE)
no_fluoridates <- draw_slope_plot(dmft_data, FALSE)

fluoridates + no_fluoridates +
  plot_annotation(
    title = "Decayed, missing, and filled teeth at age 12, by country",
    caption = "Note: Data is based on national oral health surveys, national health reports, national registers, personal communication and scientific journals.<br><br>
Source: Malmö University's Oral Health Country/Area Profile Project (CAPP)<br>
Graphic: Pat Schloss (after Soph Warnes, CNN)",
    theme = theme(
      plot.title = element_text(face = "bold", size = 9,
                                margin = margin(t = 2)),
      plot.caption = element_textbox_simple(hjust = 0, color = "darkgray",
                                            size = 7,
                                            margin = margin(t = 15)),
      plot.margin = margin(t = 5, l = 2, r = 2, b = 5)
    )
  )

ggsave("fluoride.png", width = 6, height = 3)
```
