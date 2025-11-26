---
layout: post
title: "How to simplify a busy set of panels from a figure in Nature in R with ggplot2 (CC386)"
blurb: "An improvement"
author: "PD Schloss"
date: 2025-11-26 12:00
comments: false
youtube: VvOwHiITfrM
start_hash: 
end_hash: 
---

Pat converts a very busy set of panels that used jittered point, bars, confidence intervals, and overly precise P-values into a simplified faceted plot in R using ggplot2. The original panels were included as part of an article published in Nature. He created the figure using R, ggplot2, ggtext, DescTools, readxl, and other tools from the tidyverse. The functions he used from these packages included aes, anova, as.character, as_tibble, bind_rows, case_when, coord_cartesian, download.file, drop_na, DunnettTest, element_blank, element_markdown, element_text, facet_grid, factor, filter, format, geom_jitter, geom_text, ggplot, ggsave, if_else, labeller, labs, library, lm, map, margin, mutate, n, names, nest, paste0, pivot_longer, pluck, position_jitter, pull, read_excel, round, scale_color_identity, scale_fill_identity, select, stat_summary, str_remove, theme, theme_classic, unit, and unnest. The newsletter describing this visualization at a 30,000 ft view can be [found here](https://shop.riffomonas.org/posts/did-you-know-you-can-do-statistics-in-r). You can find the original article presenting the figure [here](https://www.nature.com/articles/s41586-025-09709-1). [Here's a video](https://youtu.be/0WMsCoJ8DKs) critiquing the original plot. If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```R
library(tidyverse)
library(readxl)
library(DescTools)
library(ggtext)

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
    color = "black",
    pretty_p = if_else(pval < 0.05, "*", ""),
    generation = factor(
      generation,
      levels = c("Parental", "LN1", "LN7", "LN8", "LN9")))
  

protein_labels <- experiment_wide %>%
  mutate(pretty_protein = paste0("**", protein, "** (", pretty_p, ")"))

labels <- pull(protein_labels, pretty_protein)
names(labels) <- pull(protein_labels, protein) %>% as.character()


protein_data %>%
  mutate(generation = factor(
    generation,
    levels = c("Parental", "LN1", "LN7", "LN8", "LN9"))) %>%
  ggplot(aes(x = relative_levels, y = replicate, fill = color, color = color)) +
  geom_jitter(
    position = position_jitter(height = 0.3, seed = 19760620),
    alpha = 0.33, shape = 21, color = "black"
  ) +
  stat_summary(geom = "crossbar", fun = "median",
               lineend = "round", linewidth = 0.4) +
  geom_text(data = dunnett_results, aes(label = pretty_p), x = 0,
            size = 20, size.unit = "pt", vjust = 0.75) +
  scale_color_identity() +
  scale_fill_identity() +
  facet_grid(generation ~ protein, scale = "free_y", space = "free_y",
             switch = "y", axes = "all_y", axis.labels = "margins",
             labeller = labeller(protein = labels)
             ) +
  coord_cartesian(
    xlim = c(0, NA),
    reverse = "y") +
  labs(
    y = NULL,
    x = "Protein level relative to parental"
  ) +
  theme_classic() +
  theme(
    strip.placement = "outside",
    strip.text.y.left = element_text(angle = 0, hjust = 0, 
                                     margin = margin(r = -10)),
    strip.text.x = element_markdown(),
    strip.background = element_blank(),
    strip.clip = "off",
    
    axis.text.y = element_text(margin = margin(r = 4)),
    axis.ticks.y = element_blank(),
    panel.spacing.y = unit(1, "pt")
  )

ggsave("lymph_node_levels.png", width = 7, height = 4)
```