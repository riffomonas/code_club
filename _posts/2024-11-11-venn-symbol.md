---
layout: post
title: "Recreating a Venn diagram with ggplot2 in R as a scatter plot (CC314)"
blurb: "Making a Venn diagram with geom_point()"
author: "PD Schloss"
date: 2024-11-11 8:00
comments: false
youtube: of6wzzQxqJg
start_hash: 
end_hash: 
---

Pat shows how to use ggplot2 to generate a Venn diagram as a scatter plot using the geom_point function. He customizes the appearance using annotate, coord_cartesian, scale_alpha_discrete, scale_fill_manual, and theme. The newsletter describing how I would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/drawing-venn-diagrams-with-ggplot2).

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)

tibble(x = c(-0.5, 0.5),
       y = c(0, 0),
       group = c("A", "B")) %>%
  ggplot(aes(x = x , y = y, fill = group, alpha = group)) + 
  geom_point(shape = 21, show.legend = FALSE,
             size = 75, stroke = 1) +
  annotate("point", x = 0.5, y = 0.0,
           size = 75, shape = 21, fill = NA, stroke = 1) +
  annotate("text", x = c(-0.75, 0, 0.75), y = 0,
           label = c("13,150", "9,296", "9,575"), fontface = "bold") +
  annotate("text", x = c(-0.75, 0.75), y = 0.55,
           label = c("RASTtk\n22,446/118,833",
                     "Pfam Database\n18,871/118,833"),
           fontface = "bold", lineheight = 0.8) +
  coord_cartesian(xlim = c(-1.3, 1.3),
                  ylim = c(-0.5, 0.75), expand = FALSE) +
  scale_alpha_discrete(breaks = c("A", "B"),
                       range = c(1, 0.25)) +
  scale_fill_manual(breaks = c("A", "B"),
                    values = c("#FE8C8D", "dodgerblue")) +
  theme_void() +
  theme(
    plot.background = element_rect(fill = "white", color = NA)
  )

ggsave("venn_symbol.png", width = 4, height = 3)
```