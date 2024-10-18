---
layout: post
title: "Converting a 1D scatter plot to a histogram or density plot using the ggplot2 package in R (CC307)"
blurb: "Working with geom_histogram() and geom_density()"
author: "PD Schloss"
date: 2024-10-17 9:00
comments: false
youtube: NYEOQcO3tbI
start_hash: 
end_hash: 
---

Pat critiques a recent figure that he recreated and creates a new plot as a histogram. Along the way, he experiments with overlaying a density plot. He customizes the appearance using the annotate, labs, scale_x_continuous, scale_y_continuous, and scale_fill_manual functions. The plot is based on simulated data for Figure 1A from the paper, "Exploring novel microbial metabolites and drugs for inhibiting *Clostridioides difficile*" by Ahmed Abouelkhair and Mohamed Seleem (https://journals.asm.org/doi/10.1128/msphere.00273-24). The newsletter describing how I would go about generating the figure can be found at https://shop.riffomonas.org/posts/building-data-visualization-intuition. 



<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```
library(tidyverse)
set.seed(19760620)

sim_data <- tibble(
  compound = 1:527,
  percent_inhibition = sample(
    c(rnorm(n = 464, mean = 0, sd = 30),
      runif(63, 91, 100))
    )
  ) %>%
  mutate(strong = percent_inhibition >= 90)

sim_data %>%
  ggplot(aes(x = percent_inhibition, fill = strong)) +
#  geom_histogram(binwidth = 7) +
  geom_histogram(breaks = seq(-100, 100, 5),
                 show.legend = FALSE) +
  annotate(geom = "text",
                x = c(0, 45),
                y = c(5, 35),
                label = c("464 compounds\nwere below 90%",
                          "63 compounds\nwere above 90%"),
                fontface = "bold",
           color = c("black", "red"),
           hjust = c(0.5, 0)) +
  labs(x = "Inhibition of bacterial growth (%)",
       y = "Number of compounds") +
  scale_y_continuous(expand = c(0, 0)) +
  scale_x_continuous(limits = c(-100, 100)) +
  scale_fill_manual(breaks = c(FALSE, TRUE),
                   values = c("gray", "red")) +
  theme_classic()

ggsave("inhibition_histogram.png", width = 6, height = 4)

# sim_data %>%
#   ggplot(aes(x = percent_inhibition, fill = strong)) +
#   geom_density()
# 
# sim_data %>%
#   ggplot(aes(x = percent_inhibition, fill = strong, color = strong)) +
#   geom_histogram(binwidth = 7, aes(y = after_stat(density))) +
#   geom_density(alpha = 0.2)
```