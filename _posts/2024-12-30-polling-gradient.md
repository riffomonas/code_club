---
layout: post
title: "Recreating gradient colored lines in R with ggforce's geom_link function (CC328)"
blurb: "Animating with R"
author: "PD Schloss"
date: 2024-12-30 8:00
comments: false
youtube: JWrO8YeblV0
start_hash: 
end_hash: 
---

Pat recreates a gradient colored barbell plot made by Nate Silver in R with the help of the ggplot2, ggforce, ggtext, and glue packages. The "handle" of this dumbbell plot was made into a gradient of colors using the geom_link function from the ggforce package. Along the way he also uses the annotate, geom_point, geom_textbox, scale_x_continuous, scale_y_continuous, coord_cartesian, scale_color_gradient, scale_fill_manual, labs, theme_manual, and theme functions. Nate Silver's newsletter can be [found on his substack](https://www.natesilver.net/p/hopium-comes-at-a-high-price). Pat's newsletter describing how he would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/visualizing-bias-in-polling-data-with-a-dumbbell-plot).

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)
library(ggtext)
library(glue)
library(ggforce)

states <- c("Arizona", "North Carolina", "Nevada", "Georgia",
  "Pennsylvania", "Wisconsin", "Michigan", "**National**")

margins <- tibble(
  state = factor(states, levels = states),
  polling = c(2.4, 1.1, 0.6, 1.0, 0.2, -1.0, -1.2, -1.0),
  actual = c(5.5, 3.2, 3.1, 2.2, 1.7, 0.9, 1.4, 1.5),
  state_index = as.numeric(state)
)

margins %>%
  pivot_longer(cols = c(polling, actual)) %>%
  mutate(pretty = if_else(value < 0,
                          glue("Harris +{abs(value)}"),
                          glue("Trump +{abs(value)}")),
         pretty = if_else(name == "actual", glue("**{pretty}**"), pretty)) %>%
  ggplot(aes(x = value, y = state_index, fill = name, color = name,
             label = pretty)) +
  
  annotate("segment", x = -2:5, xend = -2:5, y = 0.5, yend = 8.08,
           linewidth = 0.25, linetype = "dotted", color = "gray") +
  annotate("segment", x = 0, xend = 0, y = 0.5, yend = 8.08,
           linewidth = 0.25, linetype = "solid", color = "black") +
  annotate("segment", y = 6:8,
           x = -0.25, xend = 0.25, color = "white", linewidth = 2.5) +
  annotate("segment", y = 5 - 0.33,
           x = -0.25, xend = 0.25, color = "white", linewidth = 4) +
  annotate("segment", x = c(-1, 1.5), xend = c(-1, 1.5), y = 8.4, yend = 8.25,
           linewidth = 0.25) +
  annotate("text", x = c(-0.95, 1.45), y = 8.5, size = 2,
           label = c("POLLING AVERGAGE MARGIN", "ACTUAL MARGIN"),
           fontface = "bold", hjust = c(1, 0)) +
  
  geom_link(data = margins, linewidth = 2.5, alpha = 1,
               mapping = aes(x = polling, xend = actual,
                             y = state_index, yend = state_index,
                             color = after_stat(index)),
               inherit.aes = FALSE,
            n = 1000,
               show.legend = FALSE) +
  geom_point(shape = 21, size = 2.5, color = "darkgray", stroke = 0.4,
             show.legend = FALSE) +
  geom_point(data = margins, aes(x = actual, y = state_index), inherit.aes = FALSE,
             shape = 21, size = 2.5, color = "#555555", fill = NA, stroke = 0.4,
             show.legend = FALSE) +
  geom_textbox(aes(y = state_index - 0.33),
            color = "black", size = 2.5,
            fill = NA, box.colour = NA, width = NULL,
            show.legend = FALSE) +
  scale_x_continuous(limits = c(-2, 6.25),
                     breaks = -2:5,
                     labels = c("+2", "Harris +1", "0",
                                "Trump +1", "+2", "+3", "+4", "+5"),
                     expand = c(0, 0)) +
  scale_y_continuous(breaks = 1:8,
                     labels = states, expand = c(0, 0)) +
  coord_cartesian(clip = "off") +
  scale_color_gradient(low = "lightgray", high = "#3CB999") +
  scale_fill_manual(breaks = c("polling", "actual"),
                     values = c("white", "#3CB999")) +
  labs(x = NULL, y = NULL,
       title = "Polls consistently underestimated Trump") +
  theme_minimal() +
  theme(
    plot.background = element_rect(fill = "white", color = "white"),
    panel.grid.minor = element_blank(),
    panel.grid.major.y = element_blank(),
    panel.grid.major.x = element_blank(),
    plot.title.position = "plot",
    plot.title = element_text(face = "bold", size = 10),
    axis.text.x = element_markdown(size = 8, color = "black"),
    axis.text.y = element_markdown(size = 9, color = "black", hjust = 0),
    
  )

ggsave("polling_bias.png", width = 5, height = 3.6)
```