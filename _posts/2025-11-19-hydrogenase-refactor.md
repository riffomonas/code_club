---
layout: post
title: "How to convert heat maps to a jittered dot plot in R using ggplot2 and patchwork (CC384)"
blurb: "So much better..."
author: "PD Schloss"
date: 2025-11-19 12:00
comments: false
youtube: QuJFET3MUqk
start_hash: 
end_hash: 
---

Pat converts a heat map to a jittered dot plot to make the visualization more interpretable in R using the ggplot2 and patchwork packages. The original heatmaps were included as part of an article published in Nature Microbiology. He created the figure using R, ggplot2, showtext, ggtext, and other tools from the tidyverse. The functions he used from these packages included aes, arrange, case_when, coord_cartesian, download.file, drop_na, element_blank, element_line, element_markdown, element_text, facet_grid, factor, filter, geom_jitter, geom_tile, geom_vline, ggplot, ggsave, if_else, inner_join, labs, library, margin, median, mutate, pivot_longer, pivot_wider, plot_layout, position_jitter, pull, read_excel, recode, rename_all, scale_fill_viridis_c, scale_x_log10, select, set.seed, stat_summary, str_detect, sum, summarize, theme, and theme_classic. The newsletter describing this visualization at a 30,000 ft view can be [found here](https://shop.riffomonas.org/posts/what-s-the-most-important-part-of-your-plot-make-sure-it-s-mapped-to-the-axes). You can find the original article presenting the [figure here](https://www.nature.com/articles/s41564-025-02154-w). [Here's a video](https://youtu.be/x7Vey8hFxW4) critiquing the original plot. If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```R
library(tidyverse)
library(readxl)
library(ggtext)
library(patchwork)

set.seed(19760620)

download.file("https://static-content.springer.com/esm/art%3A10.1038%2Fs41564-025-02154-w/MediaObjects/41564_2025_2154_MOESM11_ESM.xlsx", "source_data_fig1.xlsx")
download.file("https://static-content.springer.com/esm/art%3A10.1038%2Fs41564-025-02154-w/MediaObjects/41564_2025_2154_MOESM4_ESM.xlsx", "supplementary_data_1.xlsx")

# metag %>%
#   mutate(cpg = if_else(cpg > 1, 1, cpg)) %>%
#   ggplot(aes(x = sample, y = gene, fill = cpg)) +
#   geom_tile() +
#   scale_fill_viridis_c(direction = -1)

metag <- read_excel("source_data_fig1.xlsx", sheet = "Fig 1a - Stool MetaG cpg",
                         col_types = c("text", "text", "numeric")) %>%
  pivot_wider(names_from = "gene", values_from = "cpg", values_fill = 0) %>%
  pivot_longer(-sample, names_to = "gene", values_to = "abundance") %>%
  mutate(gene = recode(
    gene,
    "NiFe" = "[NiFe]-hydrogenase",
    "FeFe" = "[FeFe]-hydrogenase",
    "Fe" = "[Fe]-hydrogenase",
    "McrA" = "Methanogenesis (*mcrA*)",
    "Mcr" = "Methanogenesis (*mcrA*)",
    "SdhA-FrdA" = "Succinogenesis (*sdhA*, *frdA*)",
    "AsrA" = "Sulfidogenesis (*asrA*)",
    "DsrA" = "Sulfidogenesis (*dsrA*)",
    "NapA" = "NO<sub>3</sub><sup>-</sup> reduction (*napA*, *narG*)",
    "NarG" = "NO<sub>3</sub><sup>-</sup> reduction (*napA*, *narG*)",
    "NirK" = "NO<sub>2</sub><sup>-</sup> reduction to NO (*nirK*, *nirS*)",
    "NirS" = "NO<sub>2</sub><sup>-</sup> reduction to NO (*nirK*, *nirS*)",
    "NorB" = "NO reduction to N2O (*norB*)",
    "NosZ" = "N<sub>2</sub>O reduction (*nosZ*)",
    "NrfA" = "NO<sub>2</sub><sup>-</sup> reduction to NH<sub>4</sub><sup>+</sup> (*nrfA*)",
    "NifH" = "N<sub>2</sub> fixation (*nifH*)",
    "CooS" = "Acetogenesis (*acsB*, *cooS*)",
    "AtpA" = "O<sub>2</sub> detoxification and/or aerobic<br>respiration (*atpA*)",
    "CydA" = "O<sub>2</sub> detoxification and/or aerobic<br>respiration (*cydA*)",
    "CcoN" = "Aerobic respiration<br>(*ccoN*, *coxA*, *cyoA*)",
    "CoxA" = "Aerobic respiration<br>(*ccoN*, *coxA*, *cyoA*)",
    "CyoA" = "Aerobic respiration<br>(*ccoN*, *coxA*, *cyoA*)")
  ) %>%
  filter(str_detect(gene, "[\\(\\[]")) %>%
  summarize(abundance = sum(abundance), .by = c(sample, gene)) %>%
  mutate(
    type = if_else(str_detect(gene, "\\["), "hydrogenase", "other"),
    dataset = "Metagenome\n(N = 100)"
  )

gene_order <- metag %>%
  summarize(median_abundance = median(abundance), .by = "gene") %>%
  arrange(median_abundance) %>%
  pull(gene)

metag_panel <- metag %>%
  mutate(gene = factor(gene, levels = gene_order)) %>%
  ggplot(aes(x = abundance, y = gene)) +
  geom_jitter(
    position = position_jitter(width = 0, height = 0.3, seed = 19760620),
    size = 0.4, color = "gray"
    ) +
  stat_summary(
    fun = median, geom = "crossbar",
    color = "dodgerblue", linewidth = 0.2, width = 0.9) +
  facet_grid(type~dataset, scales = "free_y", space = "free_y") +
  labs(x = "CPG", y = NULL)


metat <- read_excel("source_data_fig1.xlsx", sheet = "Fig 1a- Stool MetaT RPKM",
                    col_types = c("text", "text", "numeric")) %>%
  pivot_wider(names_from = "gene",
              values_from = "total.RPKM",
              values_fill = 0) %>%
  pivot_longer(-sample, names_to = "gene", values_to = "abundance") %>%
  mutate(gene = recode(
    gene,
    "NiFe" = "[NiFe]-hydrogenase",
    "FeFe" = "[FeFe]-hydrogenase",
    "Fe" = "[Fe]-hydrogenase",
    "McrA" = "Methanogenesis (*mcrA*)",
    "Mcr" = "Methanogenesis (*mcrA*)",
    "SdhA-FrdA" = "Succinogenesis (*sdhA*, *frdA*)",
    "AsrA" = "Sulfidogenesis (*asrA*)",
    "DsrA" = "Sulfidogenesis (*dsrA*)",
    "NapA" = "NO<sub>3</sub><sup>-</sup> reduction (*napA*, *narG*)",
    "NarG" = "NO<sub>3</sub><sup>-</sup> reduction (*napA*, *narG*)",
    "NirK" = "NO<sub>2</sub><sup>-</sup> reduction to NO (*nirK*, *nirS*)",
    "NirS" = "NO<sub>2</sub><sup>-</sup> reduction to NO (*nirK*, *nirS*)",
    "NorB" = "NO reduction to N2O (*norB*)",
    "NosZ" = "N<sub>2</sub>O reduction (*nosZ*)",
    "NrfA" = "NO<sub>2</sub><sup>-</sup> reduction to NH<sub>4</sub><sup>+</sup> (*nrfA*)",
    "NifH" = "N<sub>2</sub> fixation (*nifH*)",
    "CooS" = "Acetogenesis (*acsB*, *cooS*)",
    "AtpA" = "O<sub>2</sub> detoxification and/or aerobic<br>respiration (*atpA*)",
    "CydA" = "O<sub>2</sub> detoxification and/or aerobic<br>respiration (*cydA*)",
    "CcoN" = "Aerobic respiration<br>(*ccoN*, *coxA*, *cyoA*)",
    "CoxA" = "Aerobic respiration<br>(*ccoN*, *coxA*, *cyoA*)",
    "CyoA" = "Aerobic respiration<br>(*ccoN*, *coxA*, *cyoA*)")
  ) %>%
  filter(str_detect(gene, "[\\(\\[]")) %>%
  summarize(abundance = sum(abundance), .by = c(sample, gene)) %>%
  mutate(
    type = if_else(str_detect(gene, "\\["), "hydrogenase", "other"),
    dataset = "Metatranscriptome\n(N = 78)"
  )

metat_panel <- metat %>%
  mutate(gene = factor(gene, levels = gene_order),
         abundance = if_else(abundance == 0, 0.005, abundance)) %>%
  ggplot(aes(x = abundance, y = gene)) +
  geom_jitter(
    position = position_jitter(width = 0, height = 0.3, seed = 19760620),
    size = 0.4, color = "gray"
  )+
  geom_vline(xintercept = 0.01, color = "gray", linewidth = 0.2) +
  stat_summary(
    fun = median, geom = "crossbar",
    color = "dodgerblue", linewidth = 0.2, width = 0.9) +
  facet_grid(type~dataset, scales = "free_y", space = "free_y") +
  scale_x_log10(
    limits = c(2.5e-3, 9e3), expand = c(0, 0),
    breaks = c(0.005, 0.1, 1, 10, 100, 1000),
    labels = c(0, "10<sup>-1</sup>", "10<sup>0</sup>", "10<sup>1</sup>", 
               "10<sup>2</sup>", "10<sup>3</sup>")
  ) +
  labs(x = "RPKM", y = NULL) +
  geom_vline(xintercept = 2.5e-3, linewidth = 0.2) +
  coord_cartesian(clip = "off")

metagt_map <- read_excel(
  "supplementary_data_1.xlsx", sheet = "Transcr-Abun FuncGenes RPKM ",
  range = "A2:B81") %>%
  drop_na() %>%
  rename_all(tolower)

ratio_panel <- inner_join(metag, metagt_map, by = c("sample" = "metag")) %>%
  inner_join(., metat, by = c("metat" = "sample", "gene")) %>%
  mutate(
    ratio = case_when(abundance.y == 0 ~ 0.2,
                      abundance.x == 0 & abundance.y != 0 ~ 7e5,
                      TRUE ~ abundance.y / abundance.x)
    ) %>%
  select(sample, gene, type = type.x, abundance.x, abundance.y, ratio) %>%
  mutate(gene = factor(gene, levels = gene_order),
         dataset = "Ratio\n(N = 78)"
  ) %>%
  ggplot(aes(x = ratio, y = gene)) +
  geom_jitter(
    position = position_jitter(width = 0, height = 0.3, seed = 19760620),
    size = 0.4, color = "gray",
  )+
  geom_vline(xintercept = c(0.6, 3e5), color = "gray", linewidth = 0.2) +
  stat_summary(
    fun = median, geom = "crossbar",
    color = "dodgerblue", linewidth = 0.2, width = 0.9) +
  facet_grid(type~dataset, scales = "free_y", space = "free_y") +
  scale_x_log10(
    limits = c(0.1, 1.8e6),
    breaks = c(0.2, 1, 10, 100, 1000, 10000, 1e5, 7e5),
    labels = c(0, "10<sup>0</sup>", "10<sup>1</sup>", "10<sup>2</sup>",
               "10<sup>3</sup>", "10<sup>4</sup>", "10<sup>5</sup>",
               "Not in<br>metag"),
    expand = c(0, 0)
  ) +
  labs(x = "RPKM/CPG", y = NULL) +
  geom_vline(xintercept = 0.1, linewidth = 0.2) +
  coord_cartesian(clip = "off")
  

metag_panel + metat_panel + ratio_panel +
  plot_layout(axes = "collect") &
  theme_classic() +
  theme(
    axis.line = element_line(linewidth = 0.2),
    axis.line.y = element_line(linewidth = 0.2),
    
    axis.ticks = element_line(linewidth = 0.2),
    
    axis.text.x = element_markdown(size = 6),
    axis.text.y = element_markdown(size = 5),
    axis.title.x = element_text(size = 7, margin = margin(t = -2)),
    
    strip.text.y = element_blank(),
    strip.background = element_blank(),
    strip.text.x = element_text(size = 7),
    
    plot.margin = margin(t = 0, r = 2, b = 2, l = 2))

ggsave("hydrogenase_refactor.png", width = 7, heigh = 3.5)
```