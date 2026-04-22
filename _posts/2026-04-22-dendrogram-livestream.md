---
layout: post
title: "Adding a dendrogram to a set of heatmaps to display genomics data in ggplot2 (CC418)"
blurb: "Pat's first dendrogram!"
author: "PD Schloss"
date: 2026-04-22 12:00
comments: false
youtube: fzogphProRQ
start_hash: 
end_hash: 
---

In this livestream, Pat recreates a set of heatmaps with a dendrogram from a paper published in the journal Nature Microbiology. He created the figure using R, readxl, dplyr, ggplot2, ggtext, showtext, patchwork, ggdendro, and other tools from the tidyverse. The functions he used from these packages included aes, as.dendrogram, as.dist, case_when, colnames, colorRampPalette, column_to_rownames, coord_cartesian, dendro_data, download.file, dup_axis, element_blank, element_markdown, element_text, factor, filter, font_add_google, function, geom_segment, geom_tile, ggplot, hclust, label, labs, length, library, margin, mutate, ncol, pivot_longer, pivot_wider, plot_annotation, plot_layout, read_excel, rename, rename_all, rev, sample_b = , scale_fill_gradientn, scale_fill_manual, scale_x_continuous, scale_y_continuous, segment, seq_along, showtext_auto, showtext_opts, sum, and unit. The original Nature Microbiology article can be [found here](https://www.nature.com/articles/s41564-026-02327-1). His critique of the visualization can be [seen here](https://www.youtube.com/watch?v=t4OWQE7mwoo). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```
library(tidyverse)
library(readxl)
library(showtext)
library(patchwork)
library(ggdendro)
library(ggtext)
library(RColorBrewer)

font_add_google("Nunito", "nunito", regular.wt = 500)
showtext_opts(dpi = 300)
showtext_auto()

xlsx_url <- "https://static-content.springer.com/esm/art%3A10.1038%2Fs41564-026-02327-1/MediaObjects/41564_2026_2327_MOESM6_ESM.xlsx"
xlsx_file <- "figure2.xlsx"
download.file(xlsx_url, xlsx_file)

info <- read_excel(xlsx_file, sheet = "fig.2a.info") %>%
  rename_all(tolower) %>% 
  mutate(mda = case_when(mda == "after" ~ "After",
                         mda == "before" ~ "Before",
                         mda == "no MDA" ~ "No MDA"))

relatedness <- read_excel(xlsx_file,  sheet = "fig.2a.matrix") %>%
  rename("sample_a" = "...1") %>%
  pivot_longer(-sample_a, names_to = "sample_b", values_to = "relatedness") %>%
  filter(sample_a %in% info$id) %>%
  filter(sample_b %in% info$id)
  
# length(info$id)
# length(relatedness$sample_a)
# ncol(relatedness)
# 
# sum(!info$id %in% relatedness$sample_a)
# sum(!info$id %in% colnames(relatedness))
# sum(!colnames(relatedness) %in% relatedness$sample_a)

subtract_one <- function(x) {1 - x}

distances <- relatedness %>%
  pivot_wider(names_from = "sample_b", values_from = "relatedness") %>%
  column_to_rownames("sample_a") %>%
  subtract_one() %>%
  as.dist()
  
clustering <- as.dendrogram(hclust(distances, method = "complete"))
ddata <- dendro_data(clustering)
branch_data <- segment(ddata)
tree_labels <- label(ddata)$label

tree <- branch_data %>%
  ggplot(aes(x = y, y = x, xend = yend, yend = xend)) +
  geom_segment(linewidth = 0.2) +
  scale_x_continuous(transform = "reverse") +
  scale_y_continuous(
    breaks = seq_along(tree_labels),
    label = tree_labels,
  ) +
  coord_cartesian(expand = FALSE) +
  labs(x = NULL, y = NULL)

r_heatmap <- relatedness %>%
  mutate(sample_a = factor(sample_a, rev(tree_labels)),
         sample_b = factor(sample_b, tree_labels)) %>%
  ggplot(aes(x = sample_a, y = sample_b, fill = relatedness)) +
  geom_tile() + 
  coord_cartesian(expand = FALSE) +
  labs(x = NULL, y = NULL) +
  scale_fill_gradientn(
    name = "Relatedness",
    limits = c(-0.05, 1.05),
    breaks = c(0, 0.5, 1),
    colors = colorRampPalette(brewer.pal(n = 7, name = "YlOrRd"))(100),
    # low = "#ffffcc",
    # high = "#800026",
    # mid = "#fd8d3c",
    # midpoint = 0.5
  ) +
  theme(
    legend.key.width = unit(11, "pt"),
    legend.key.height = unit(12, "pt")
  )



mda <- info %>%
  mutate(id = factor(id, rev(tree_labels))) %>%
  ggplot(aes(x = id, y = 1, fill = mda)) +
  geom_tile() + 
  coord_cartesian(expand = FALSE) +
  labs(x = NULL, y = "MDA") +
  scale_y_continuous(sec.axis = dup_axis(breaks = NULL, labels = NULL)) +
  scale_fill_manual(
    name = "MDA", 
    breaks = c("After", "Before", "No MDA"),
    values = c("#56B4E9", "#D45E00", "#D3D3D3")
  ) 
  
year <- info %>%
  mutate(id = factor(id, rev(tree_labels))) %>%
  ggplot(aes(x = id, y = 1, fill = as.character(year.clean))) +
  geom_tile() + 
  coord_cartesian(expand = FALSE) +
  labs(x = NULL, y = "Year") +
  scale_y_continuous(sec.axis = dup_axis(breaks = NULL, labels = NULL)) +
  scale_fill_manual(
    name = "Year", 
    breaks = c("2016", "2017", "2018", "2019", "2020"),
    values = c("#C7E9B4", "#7FCDBB", "#40B6C4", "#2C7FB8", "#253493")
  ) 

kelch <- info %>%
  mutate(id = factor(id, rev(tree_labels))) %>%
  ggplot(aes(x = id, y = 1, fill = kelch13.clean)) +
  geom_tile() + 
  coord_cartesian(expand = FALSE) +
  labs(x = NULL, y = "*kelch13*") +
  scale_y_continuous(sec.axis = dup_axis(breaks = NULL, labels = NULL)) +
  scale_fill_manual(
    name = "*kelch13*", 
    values = c("C580Y" = "#FF0100", "F446I" = "#90ED91", "G449A" = "#0272B2",
               "M476I" = "#019E73", "M562I" = "#E69F00", "P441L" = "#A01FF0",
               "R561H" = "#FFDB6D", "WT" = "#ACD8E6")
  )

layout <- "
  #A
  #B
  #C
  DE
"

combined <- mda + theme(plot.margin = margin(t = 1, r = 0, b = 1, l = 0)) +
  year + theme(plot.margin = margin(t = 1, r = 0, b = 1, l = 0)) +
  kelch + theme(plot.margin = margin(t = 1, r = 0, b = 1, l = 0)) +
  tree + theme(plot.margin = margin(t = 1, r = 0, b = 1, l = 0)) +
  r_heatmap + theme(plot.margin = margin(t = 1, r = 0, b = 1, l = 0)) +
  plot_layout(axes = "collect", guides = "collect", design = layout,
              widths = c(0.1, 0.9),
              heights = c(0.03, 0.03, 0.03, 0.91)) +
  plot_annotation(
    theme = theme(plot.margin = margin(2, 2, 2, 2))) &
  theme(
    text = element_text(family = "nunito"),
    axis.text = element_blank(),
    axis.ticks = element_blank(),
    axis.title = element_blank(),
    axis.title.y.right = element_markdown(angle = 0, vjust = 0.5, hjust = 0,
                                          size = 8.5),
    legend.title = element_markdown(size = 8, margin = margin(b = 2)),
    legend.text = element_text(size = 8),#, margin = margin(l = 2, b = 1)),
    legend.key.spacing.y = unit(2, "pt"),
    legend.key.size = unit(7, "pt"),
    legend.spacing.y = unit(20, "pt"),
    legend.justification = c(0, 0.5),
    legend.margin = margin(0, 0, 0, 0),
    legend.box.spacing = unit(-20, "pt")
  )

ggsave("figure2.png", combined, width = 7.5, height = 6.75) #height = 8.33)
```