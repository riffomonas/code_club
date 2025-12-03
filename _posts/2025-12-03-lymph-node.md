---
layout: post
title: "How to combine map and patchwork to make facets of facets with ggplot2 in R (CC387)"
blurb: "Soooo cool!"
author: "PD Schloss"
date: 2025-12-03 12:00
comments: false
youtube: nf34N5y2TOw
start_hash: 
end_hash: 
---

Pat refactors a set of panels from a horizontal to vertical orientation using a combination of facet_wrap and patchwork with the help of the map function. The original panels were included as part of an article published in Nature. He created the figure using R, dplyr, ggplot2, ggtext, DescTools, readxl, patchwork, colorspace, and other tools from the tidyverse. The functions he used from these packages included DunnettTest, aes, anova, as.character, as_factor, as_tibble, bind_rows, case_when, coord_cartesian, download.file, drop_na, element_blank, element_text, f_else, facet_wrap, factor, filter, format, function, geom_jitter, geom_richtext, geom_text, ggplot, ggsave, if_else, labs, library, lighten, lm, map, map2, margin, max, mutate, n, names, nest, paste0, pivot_longer, plot_annotation, plot_layout, pluck, position_jitter, pull, read_excel, round, scale_color_identity, scale_fill_identity, select, stat_summary, str_remove, theme, theme_classic, unit, unnest, and wrap_plots. The newsletter describing this visualization at a 30,000 ft view can be [found here](https://shop.riffomonas.org/posts/did-you-know-you-can-do-statistics-in-r). You can find the original article presenting the figure [here](https://www.nature.com/articles/s41586-025-09709-1). [Here's a video](https://youtu.be/0WMsCoJ8DKs) critiquing the original plot. If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```R
library(tidyverse)
library(readxl)
library(DescTools)
library(ggtext)
library(patchwork)
library(colorspace)

download.file(
  "https://static-content.springer.com/esm/art%3A10.1038%2Fs41586-025-09709-1/MediaObjects/41586_2025_9709_MOESM5_ESM.xlsx",
  "fig1_data.xlsx"
)

e <- read_excel("fig1_data.xlsx", sheet = "Fig. 1e", range = "A3:J33") %>%
  mutate(replicate = 1:n(), .before = "B16-F0") %>%
  pivot_longer(-replicate, names_to = "line", values_to = "relative_levels")

g <- read_excel("fig1_data.xlsx", sheet = "Fig. 1g", range = "A3:J33") %>%
  mutate(replicate = 1:n(), .before = "B16-F0") %>%
  pivot_longer(-replicate, names_to = "line", values_to = "relative_levels")

i <- read_excel("fig1_data.xlsx", sheet = "Fig. 1 i", range = "A3:J10") %>%
  mutate(replicate = 1:n(), .before = "B16-F0") %>%
  pivot_longer(-replicate, names_to = "line", values_to = "relative_levels")

k <- read_excel("fig1_data.xlsx", sheet = "Fig. 1k", range = "A3:J18") %>%
  mutate(replicate = 1:n(), .before = "B16-F0") %>%
  pivot_longer(-replicate, names_to = "line", values_to = "relative_levels")

protein_data <- bind_rows("GCLC" = e, "FSP1" = g, "GPX4" = i, "ACSL4" = k, .id = "protein") %>%
  drop_na() %>%
  select(-replicate) %>%
  mutate(
    generation = if_else(line == "B16-F0",
                         "Parental",
                         str_remove(line, " .*")
                         ),
    color = case_when(generation == "Parental" ~ "black",
                      generation == "LN1" ~ "#FE3FFF",
                      TRUE ~ "#068E00"),
    fill = lighten(color, 0.6),
    
    protein = factor(protein, levels = c("GCLC", "FSP1", "GPX4", "ACSL4"))
  ) %>%
  nest(data = -c(generation, line)) %>%
  mutate(
    replicate = LETTERS[1:n()], .by = c(generation)
  ) %>%
  unnest(data) %>%
  mutate(replicate = if_else(generation %in% c("LN1", "Parental"),
                             "",
                             replicate)
         )

experiment_wide <-  protein_data %>%
  nest(data = -protein) %>%
  mutate(experiment_p = map(data, \(x){
    
    lm(relative_levels~line, data = x) %>%
      anova() %>%
      pluck("Pr(>F)") %>%
      .[[1]]
    
  })) %>%
  unnest(experiment_p) %>%
  mutate(pretty_p = if_else(experiment_p < 1e-3,
                            "P < 0.001",
                            format(round(experiment_p, digits = 2)))) %>%
  select(protein, pretty_p)



dunnett_results <- protein_data %>%
  filter(generation != "LN1") %>%
  nest(data = -protein ) %>%
  mutate(dunnett = map(data, \(x){
    
    DunnettTest(relative_levels ~ line, data = x) %>%
      pluck("B16-F0") %>%
      as_tibble(rownames = "line") %>%
      mutate(line = str_remove(line, "-B16-F0")) %>%
      select(line, pval)
    
  })) %>%
  unnest(dunnett) %>%
  mutate(
    generation = if_else(line == "B16-F0",
                       "Parental",
                       str_remove(line, " .*")
                       )
    ) %>%
  nest(data = -c(generation, line)) %>%
  mutate(
    replicate = LETTERS[1:n()], .by = c(generation)
  ) %>%
  unnest(data) %>%
  mutate(
    color = "black", fill = NA,
    pretty_p = if_else(pval < 0.05, "*", ""),
    generation = factor(
      generation,
      levels = c("Parental", "LN1", "LN7", "LN8", "LN9")))
  

protein_labels <- experiment_wide %>%
  mutate(pretty_protein = paste0("**", protein, "**<br>(", pretty_p, ")"),
         relative_levels = 5.75,
         replicate = "",
         generation = as_factor("Parental"),
         color = "black", fill = NA)

labels <- pull(protein_labels, pretty_protein)
names(labels) <- pull(protein_labels, protein) %>% as.character()

plotting_function <- function(p, d){
  
  dun_test <- dunnett_results %>%
    filter(protein == p)
  
  panel_label <- protein_labels %>%
    filter(protein == p)
    
    
  d %>%
  ggplot(aes(x = relative_levels, y = replicate, fill = fill, color = color)) +
    geom_jitter(
      position = position_jitter(height = 0.25, seed = 19760620),
      stroke = 0.2,
      # alpha = 0.33, 
      shape = 21, color = "black"
    ) +
    stat_summary(geom = "crossbar", fun = "median",
                 lineend = "round", linewidth = 0.4) +
    geom_text(data = dun_test, aes(label = pretty_p), x = 5.5,
              size = 20, size.unit = "pt", vjust = 0.75) +
    geom_richtext(
      data = panel_label, aes(label = pretty_protein), fill = NA,
      label.colour = NA, label.padding = unit(0, "lines"), hjust = 1,
      size = 3
    ) +
    scale_color_identity() +
    scale_fill_identity() +
    facet_wrap(generation ~ ., ncol = 1, scale = "free_y", space = "free_y",
               strip.position = "left"
               # switch = "y", axes = "all_y", axis.labels = "margins",
               # labeller = labeller(protein = labels)
    ) +
    coord_cartesian(
      xlim = c(0, max(max(protein_data$relative_levels), 5.5)),
      clip = "off",
      reverse = "y") +
    scale_x_continuous(
      breaks = 0:5
    ) +
    labs(
      y = NULL,
      x = "Protein level relative to parental"
    ) +
    theme_classic() +
    theme(
      strip.placement = "outside",
      strip.text.y.left = element_text(angle = 0, hjust = 0, 
                                       margin = margin(r = -10)),
      strip.background = element_blank(),
      strip.clip = "off",
      
      panel.background = element_blank(),
      
      axis.text.y = element_text(margin = margin(r = 4)),
      axis.text.x = element_text(size = 8),
      axis.ticks.y = element_blank(),
      axis.title.x = element_text(size = 10),
      panel.spacing.y = unit(1, "pt")
    )
}

protein_data %>%
  mutate(generation = factor(
    generation,
    levels = c("Parental", "LN1", "LN7", "LN8", "LN9"))) %>%
  nest(data = -protein) %>%
  mutate(plots = map2(protein, data, plotting_function)) %>%
  pull(plots) %>%
  wrap_plots(ncol = 1) +
  plot_layout(axes = "collect") +
  plot_annotation(theme = theme(plot.margin = margin(b = 0)))

ggsave("lymph_node_levels.png", width = 3.5, height = 6)
```
