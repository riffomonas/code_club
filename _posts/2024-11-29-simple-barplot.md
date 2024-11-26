---
layout: post
title: "Going for a simple with ggplot2 and dplyr (CC320)"
blurb: "Dot plot with error bars"
author: "PD Schloss"
date: 2024-11-29 8:00
comments: false
youtube: QP-ZL080IH0
start_hash: 
end_hash: 
---

Pat shows how to you can use some simple dplyr and ggplot2 functions to make a simple, but elegant and impactful figure. He uses factor and group_by, summarize, mutate from dplyr and geom_hline, geom_errorbar, geom_point, scale_y_log10, scale_color_manual, labs, theme_classic, and theme from ggplot2. His goal was to show you the minimal elements you would need to convert the default visual into something that would be publication quality. The newsletter describing how I would go about generating the figure can be found [here](https://shop.riffomonas.org/posts/some-thoughts-on-how-to-recreate-a-descending-bar-plot-with-ggplot2).

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```
library(tidyverse)
library(ggtext)

set.seed(19760620)

spore_data <- tibble(
  fraction_alive = abs(c(
    rnorm(6, 1e-1, 7e-2),
    rnorm(3, 1e-4, 7e-5), rnorm(3, 1e-3, 7e-4),
    rnorm(6, 1e-6, 7e-7),
    rnorm(6, 1e-8, 7e-9)
  )),
  heat = rep(rep(c("before heat", "after heat"), each = 3), 4),
  treatment = rep(c("A", "B", "C", "D"), each = 6)
)

spore_data %>%
  group_by(heat, treatment) %>%
  summarize(mean_alive = mean(fraction_alive),
            sd_alive = sd(fraction_alive), .groups = "drop") %>%
  mutate(heat = factor(heat,
                       levels = c("before heat",  "after heat"),
                       label = c("Before heat",  "After heat")),
         treatment = factor(
                        treatment,
                        labels = c("Valine\nNisin\nNo pressure",
                                   "Valine\nNisin\n150 MPa",
                                   "Valine\nNisin\n550 MPa",
                                   "TSB\nNisin\n550 MPa\n80\u00B0C\n37\u00B0C\n550 MPa")
                        )) %>%
  ggplot(aes(x = treatment, y = mean_alive, color = heat,
             ymin = mean_alive - sd_alive, ymax = mean_alive + sd_alive)) +
  geom_hline(yintercept = 1e-8, color = "gray", linewidth = 0.25, linetype = "dashed") +
  geom_errorbar(width = 0.3, position = position_dodge(width = 0.5),
                show.legend = FALSE) +
  geom_point(size = 3, position = position_dodge(width = 0.5)) +
  scale_y_log10(limits = c(1e-9, 1),
                breaks = c(1e0, 1e-2, 1e-4, 1e-6, 1e-8),
                labels = c("10<sup>0</sup>", "10<sup>-2</sup>", 
                           "10<sup>-4</sup>", "10<sup>-6</sup>",
                           "10<sup>-8</sup>")) +
  scale_color_manual(values = c("dodgerblue", "darkred")) +
  labs(x = NULL, y = "Fraction of <i>Bacillus</i> remaining", color = NULL) +
  theme_classic() +
  theme(
    axis.text.y = element_markdown(color = "black"),
    axis.text.x = element_text(color = "black"),
    axis.title.y = element_markdown(),
    legend.position = "inside",
    legend.position.inside = c(0.75, 0.90),
    legend.background = element_rect(color = "black"),
    legend.margin = margin(3, 3, 3, 3),
    legend.key.size = unit(10, "pt")
  )

ggsave("fraction_alive.png", width = 4, height = 4)
```