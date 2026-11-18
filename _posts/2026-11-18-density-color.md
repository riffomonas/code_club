---
layout: post
title: "Representing density with color in jitter plots with R's ggplot2 (CC439)"
blurb: "Finding problems in other people's packages"
author: "PD Schloss"
date: 2026-11-18 09:00
comments: false
youtube: Rzw4kKkhUik
start_hash: 
end_hash: 
---

The article discussed: https://www.nature.com/articles/s41556-026-01943-7

Code from this livestream: https://riffomonas.org/code_club/2026-11-18-density-color

Related critique: https://youtu.be/8K0kuriEIpU

Sign up for a complimentary consultation to help me learn your needs: https://calendly.com/pat-riffomonas/30min
My newsletter: https://shop.riffomonas.org/youtube 
Recorded workshops: https://riffomonas.org/workshops/
If you want to cite this video: https://journals.asm.org/doi/10.1128/mra.01310-22

* packages: tidyverse, readxl, ggpointdensity

* functions: abs, aes, annotate, as.numeric, bind_rows, case_when, coord_cartesian, drop_na, element_blank, element_line, element_text, facet_wrap, factor, filter, function, geom_density, geom_jitter, geom_pointdensity, ggplot, ggsave, if_else, labs, library, map, mutate, n, nest, position_jitter, read_excel, scale_color_viridis_c, select, sum, theme, unit, unnest

This work was supported, in part, by a grant from the National Institutes of Health (R25GM166785)

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



```R
library(tidyverse)
library(readxl)
library(ggpointdensity)

full_data <- read_excel("https://www.riffomonas.org/code_club/data/assets/2026-11-18-melanocytes.xlsx",
                        skip = 1, guess_max = 56000) %>%
  select(linked = Clonality, expressed = Expressed,
         classification = Call, shared = Also_in, vaf = MAF) %>%
  mutate(index = 1:n())


all <- full_data %>%
  mutate(category = "all",
         facet = "types") %>%
  select(category, vaf, facet)

linked <- full_data %>%
  select(index, linked, vaf) %>%
  mutate(linked = case_when(linked == "Clonal" ~ "linked",
                            linked == "Subclonal" ~ "not linked",
                            TRUE ~ NA),
         facet = "types") %>%
  drop_na() %>%
  select(category = linked, vaf, facet)
  
expressed <- full_data %>%
  select(index, expressed, vaf) %>%
  drop_na() %>%
  mutate(category = if_else(expressed == "Yes", 
                            "expressed", "not expressed"),
         facet = "types") %>%
  select(category, vaf, facet)

shared <- full_data %>%
  select(index, shared, vaf) %>%
  drop_na() %>%
  mutate(category = "shared",
         facet = "types") %>%
  select(category, vaf, facet)

classification <- full_data %>%
  select(index, classification, vaf) %>%
  mutate(
    category = case_when(
      classification == "Inferred Artifact" ~ "artifact",
      classification == "Inferred Mutation" ~ "mutation",
      classification == "Invalidated" ~ "invalidated",
      classification == "Validated" ~ "validated",
      TRUE ~ NA),
    facet = case_when(
      category == "artifact" ~ "artifact",
      category == "mutation" ~ "mutation",
      category == "invalidated" ~ "artifact",
      category == "validated" ~ "mutation")
    ) %>%
  select(category, vaf, facet)


count_neighbors <- function(data, threshold = 0.1){
  
  data %>%
    mutate(n_neighbors = map_int(.x = vaf,
                                 ~sum(abs(vaf - .x) < threshold) - 1))

}

long_data <- bind_rows(all, linked, expressed, shared, classification) %>%
  nest(data = -c(facet, category)) %>%
  # filter(category == "shared") %>%
  mutate(n_neighbors = map(data, count_neighbors, threshold = 0.1)) %>%
  unnest(n_neighbors) %>%
  select(category, facet, vaf, n_neighbors) %>%
  mutate(category = factor(category, 
                           levels = c("shared",
                                      "not expressed", "expressed", 
                                      "not linked", "linked",  
                                      "all",
                                      "invalidated", 
                                      "validated", "mutation", 
                                      "artifact"),
                           labels = c("Shared w/\nother cells",
                                      "Not expressed", "Expressed", 
                                      "Not linked", "Linked",  
                                      "All variants",
                                      "Invalidated", 
                                      "Validated", "Inferred", 
                                      "Inferred")),
         facet = factor(facet,
                        levels = c("types", "mutation", "artifact"),
                        labels = c("Type of variants", "Mutations",
                                   "Artefacts")))

# long_data %>%
#   ggplot(aes(x = vaf)) +
#   geom_density() +
#   facet_wrap(~category)

long_data %>%
  ggplot(aes(y = category, x = vaf, color = n_neighbors)) +
  geom_jitter(size = 0.1) +
  scale_color_viridis_c(
    breaks = c(500, 23500),
    labels = c("Low", "High")
  ) +
  facet_wrap(~facet, ncol = 1, scales = "free_y", space = "free_y",
            strip.position = "left") +
  labs(x = "Variant allelic frequency", y = NULL, color = "Variant\ndensity") +
  coord_cartesian(xlim = c(0, 1), expand = FALSE, clip = FALSE) +
  annotate(geom = "segment", layout = 1,
           x = -0.285, y = 0.5, yend = 6.5) +
  annotate(geom = "segment", layout = 2,
           x = -0.285, y = 0.5, yend = 2.5) +
  annotate(geom = "segment", layout = 3,
           x = -0.285, y = 0.5, yend = 2.5) +
  theme(
    strip.placement = "outside",
    panel.spacing.y = unit(c(10, 3), "pt"),
    
    # legend.text = element_blank(),
    legend.ticks = element_blank(),
    legend.title.position = "bottom",
    legend.title = element_text(hjust = 0.5, size = 8),
    legend.text = element_text(size = 7),
    legend.position = "right",
    legend.justification.right = "top",
    legend.key.width = unit(13, "pt"),
    
    axis.text = element_text(color = "black"),
    axis.ticks.y = element_blank(),
    axis.ticks.x = element_line(color = "gray80", linewidth = 0.2),
    panel.grid.minor = element_blank(),
    panel.grid.major.y = element_blank(),
    panel.grid.major.x = element_line(color = "gray80", linewidth = 0.2),
    panel.background = element_blank(),
    
    strip.background = element_blank()
  )

# long_data %>%
#   ggplot(aes(y = as.numeric(category), x = vaf)) +
#   geom_pointdensity(size = 0.1, method='neighbors',
#                     position = position_jitter()) +
#   scale_color_viridis_c()

# long_data %>%
#   ggplot(aes(x = as.numeric(category), y = vaf)) +
#   geom_pointdensity(size = 0.1, method='neighbors',
#                     position = position_jitter()) +
#   scale_color_viridis_c()

ggsave("figure_1.png", width = 5, height = 3.8)
```