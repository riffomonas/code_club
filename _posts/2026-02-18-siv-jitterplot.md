---
layout: post
title: "Refactoring a plot to better align with the text and make interpretation easier for readers (CC400)"
blurb: "Make it easy for your audience"
author: "PD Schloss"
date: 2026-02-18 12:00
comments: false
youtube: tW7NsqYNrUM
start_hash: 
end_hash: 
---

In this livestream, Pat refactors a plot from a paper recently published in Nature Microbiology. The figure was structured differently from how the authors described it in their text. Once the structure was made to parallel the text it was far easier to interpret. Pat makes the restructured plot in this livestream. He created the figure using R, dplyr, ggplot2, ggtext, glue, readxl, and other tools from the tidyverse. The functions he used from these packages included annotate, bind_rows, download.file, element_blank, element_line, element_markdown, element_rect, expand.grid, expansion, facet_grid, factor, geom_hline, geom_jitter, ggplot, ggsave, glue, inner_join, labs, library, map_dbl, margin, median, mutate, pivot_longer, position_jitter, pull, read_excel, replace_na, runif, scale_fill_manual, scale_y_log10, separate_wider_delim, seq, stat_summary, theme, and unit. The original Nature Microbiology article can be [found here](https://www.nature.com/articles/s41564-025-02258-3). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```R
library(tidyverse)
library(readxl)
library(glue)
library(ggtext)

data_url <- "https://static-content.springer.com/esm/art%3A10.1038%2Fs41564-025-02258-3/MediaObjects/41564_2025_2258_MOESM4_ESM.xlsx"
download.file(url = data_url, destfile = "figure1_data.xlsx")

times <- c("12 dpi",	"21 wpi",	"66 wpi")
sites <- c("Peripheral<br>Lymph Node", "Mesenteric<br>Lymph Node", "Spleen",
           "Bone<br>Marrow", "Duodenum", "Rectum", "Liver")

sites_times <- expand.grid(time = times, site = sites) %>%
  mutate(pretty_label = glue("{site}_{time}")) %>%
  pull(pretty_label)

dna <- inner_join(read_excel("figure1_data.xlsx", sheet = "Figure 1d",
                              range = "B5:E28", 
                              col_names = c("animal_id",
                                            glue("Peripheral<br>Blood_{times}")),
                              na = "ND"),
                   read_excel("figure1_data.xlsx", sheet = "Figure 1e",
                              range = "B5:W28",
                              col_names = c("animal_id", sites_times),
                              na = "ND"),
                   by = "animal_id")

rna <- inner_join(read_excel("figure1_data.xlsx", sheet = "Figure 1d",
                             range = "B32:E55", 
                             col_names = c("animal_id",
                                           glue("Peripheral<br>Blood_{times}")),
                             na = "ND"),
                  read_excel("figure1_data.xlsx", sheet = "Figure 1e",
                             range = "B33:W56",
                             col_names = c("animal_id", sites_times),
                             na = "ND"),
                  by = "animal_id")

bind_rows("SIV DNA per 10^6^ cells" = dna,
          "SIV RNA per 10<sup>6</sup> cells" = rna, .id = "nucleic_acid") %>%
  pivot_longer(cols = -c(animal_id, nucleic_acid), names_to = "site_time",
               values_to = "density") %>%
  separate_wider_delim(site_time, delim = "_", names = c("site", "time")) %>%
  mutate(
    site = factor(site, levels = c("Peripheral<br>Blood", sites)), 
    density = map_dbl(density, ~replace_na(.x, runif(1, 2e-2, 4e-2)))
    ) %>%
  ggplot(aes(x = time, y = density, fill = time)) +
  geom_jitter(shape = 21, size = 1.5, stroke = 0.1,
              position = position_jitter(width = 0.3, seed = 19760620)) +
  stat_summary(fun = median, geom = "crossbar", linewidth = 0.3, width = 1.1,
               show.legend = FALSE) +
  geom_hline(yintercept = 1e-1, color = "gray", linewidth = 0.2) +
  annotate(geom = "text",
          label = "LOD", x = 0.5, y = 1e-1,
          color = "gray", size = 7, size.unit = "pt", hjust = 0, vjust = -0.2,
          layout = c(1, 9)) +
  facet_grid(nucleic_acid~site, switch = "y",
             scale = "free_y", space = "free_y", axes = "all_x") +
  scale_y_log10(
    breaks = 10^seq(-1, 7, 1),
    labels = glue("10<sup>{seq(-1, 7, 1)}</sup>"),
    limit = c(1e-2, NA),
    expand = expansion(mult = c(0, 0.08))
  ) +
  scale_fill_manual(
    values = c("12 dpi" = "#deebf7", "21 wpi" = "#9ecae1", "66 wpi" = "#3182bd")
  ) +
  labs(x = NULL, y = NULL, fill = NULL) +
  theme(
    panel.background = element_blank(),
    panel.grid = element_blank(),
    panel.spacing.x = unit(3, "pt"),
    axis.line = element_line(linewidth = 0.2),
    axis.text.x = element_blank(),
    axis.ticks.x = element_blank(),
    axis.ticks.y = element_line(linewidth = 0.2),
    axis.text.y = element_markdown(size = 7),
    legend.position = "bottom",
    # legend.background = element_rect(fill = "gray"),
    legend.key.height = unit(8, "pt"),
    legend.key.width = unit(4, "pt"),
    legend.text = element_text(margin = margin(l = 3, r = 7)),
    legend.margin = margin(0, 0, 0, 0),
    legend.box.spacing = unit(3, "pt"),
    strip.placement = "outside",
    strip.text.y.left = element_markdown(margin = margin(0, 0, 0, 0)),
    strip.text.x = element_markdown(size = 7),
    strip.clip = "off",
    strip.background = element_blank()
  )

ggsave("figure_1d.png", width = 5.5, height = 4.4)
```