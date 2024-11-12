---
layout: post
title: "Recreating a Venn diagram with geom_ribbon from ggplot2 in R (CC315)"
blurb: "Making a Venn diagram with geom_ribbon()"
author: "PD Schloss"
date: 2024-11-14 8:00
comments: false
youtube: WGpy-gz5_LA
start_hash: 
end_hash: 
---

Pat shows how to use the geom_ribbon() function from ggplot2 to generate a Venn diagram using a function to generate a tibble representing a circle. He customizes the appearance using annotate, coord_equal, scale_alpha_discrete, scale_fill_manual, and theme. The newsletter describing how I would go about generating the figure can be found at https://shop.riffomonas.org/posts/drawing-venn-diagrams-with-ggplot2.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)

circle <- function(center_x, center_y, radius, label = "", resolution = 1000){

# x^2 + y^2 = r^2
# y^2 = r^2 - x^2
# y = +/- sqrt(r^2 - x^2)

  x <- seq(-radius, radius, length.out = resolution)
  y_upper <- sqrt(radius^2 - x^2)
  y_lower <- -sqrt(radius^2 - x^2)
  
  tibble(x = x + center_x,
         ymin = y_lower + center_y,
         ymax = y_upper + center_y,
         label = label)
  
}

bind_rows(circle(0, 0, 1, "A"),
          circle(1, 0, 1, "B")) %>%
  ggplot(aes(x = x, ymin = ymin, ymax = ymax,
             group = label, fill = label, alpha = label)) +
  geom_ribbon(color = "black", linewidth = 1, show.legend = FALSE) +
  annotate("text", x = c(-0.5, 0.5, 1.5), y = 0,
           label = c("13,150", "9,296", "9,575"), fontface = "bold") +
  annotate("text", x = c(-0.5, 1.5), y = 1.4,
           label = c("RASTtk\n22,446/118,833",
                     "Pfam Database\n18,871/118,833"),
           fontface = "bold", lineheight = 0.8, vjust = 1) +
  coord_equal() +
  scale_fill_manual(breaks = c("A", "B"),
                    values = c("#FE8C8D", "dodgerblue")) +
  scale_alpha_discrete(breaks = c("A", "B"),
                       range = c(1, 0.25)) +
  theme_void() +
  theme(
    plot.background = element_rect(fill = "white", color = NA)
  )

ggsave("venn_formula.png", width = 4, height = 3)
```