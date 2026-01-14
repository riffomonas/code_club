---
layout: post
title: "Creating and refactoring faceted petal plots in R with ggplot2 (CC393)"
blurb: "Not a fan of this petal plot"
author: "PD Schloss"
date: 2026-01-14 12:00
comments: false
youtube: 9OR9lueCuyE
start_hash: 
end_hash: 
---

Pat creates two plots in this livestream. First, he recreates a set of panels that were presented as petal plots and then he refactors those plots as dot plots starting with the raw data. The original panels were included as part of an article published in Nature Microbiology. He created the figure using R, dplyr, ggplot2, ggtext, readxl, glue, broom, and other tools from the tidyverse. The functions he used from these packages included aes, as.numeric, coord_radial, download.file, drop_na, element_blank, element_line, element_markdown, element_text, facet_grid, facet_wrap, factor, geom_abline, geom_col, geom_errorbar, geom_hline, geom_point, ggplot, ggsave, glue, guide_axis_theta, guides, labs, library, map_dfr, margin, matches, mutate, nest, p.adjust, pivot_longer, position_dodge, read_excel, rename, scale_color_manual, scale_shape_manual, scale_x_discrete, scale_y_continuous, select, str_remove, t.test, theme, theme_classic, tidy, toupper, unit, and unnest. The code that Pat generated in this episode can be found [here](https://riffomonas.org/code_club/2026-01-14-petal-plot-redo). The newsletter describing this visualization at a 30,000 ft view can be found [here](https://shop.riffomonas.org/posts/what-do-you-think-of-petal-plots-how-would-you-make-one-in-r). You can find the original article presenting the figure at [*Nature Microbiology*](https://www.nature.com/articles/s41586-025-09898-9). Here's a [video critiquing](https://youtu.be/k5RJz4MU6NQ) the original figure. If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```R
library(tidyverse)
library(readxl)
library(glue)
library(broom)
library(ggtext)

petal_data <- "https://static-content.springer.com/esm/art%3A10.1038%2Fs41586-025-09898-9/MediaObjects/41586_2025_9898_MOESM9_ESM.xlsx"

download.file(petal_data, "petal_data.xlsx")

read_excel("petal_data.xlsx") %>%
  rename(cell_line = 'cell line') %>%
  mutate(mean = 2 + mean,
         ci_upper = 2 + ci_upper,
         ci_lower = 2 + ci_lower) %>%
  ggplot(aes(x = tissue, y = mean, fill = gene,
             ymin = ci_lower, ymax = ci_upper)) +
  geom_col(show.legend = FALSE, width = 0.5) +
  geom_errorbar(width = 0, linewidth = 0.3) +
  facet_grid(cell_line~gene, switch = "y") +
  scale_x_discrete(expand = c(1/12, 1/12)) +
  scale_y_continuous(expand = c(0, 0), breaks = c(0, 1, 2)) +
  coord_radial(start = -2*pi/12) +
  theme(
    panel.grid.minor = element_blank(),
    panel.grid.major = element_line(color = 'gray', linewidth = 0.3),
    panel.background = element_blank(),
    axis.text.y = element_blank(),
    axis.text.x = element_text(size = 7, margin = margin(b = 0)),
    axis.ticks = element_blank(),
    axis.title = element_blank(),
    strip.text.y.left = element_text(angle = 0, hjust = 1),
    strip.background = element_blank(),
    panel.spacing.x = unit(10, "pt"),
    panel.spacing.y = unit(20, "pt")
  ) +
  guides(theta = guide_axis_theta(angle = 0))

ggsave("metastisis_petal_plot.png", width = 7.5, height = 4)  


raw_mestastisis_data <- "https://static-content.springer.com/esm/art%3A10.1038%2Fs41586-025-09898-9/MediaObjects/41586_2025_9898_MOESM20_ESM.xlsx"

download.file(raw_mestastisis_data, "raw_mestastisis_data.xlsx")

read_excel("raw_mestastisis_data.xlsx", sheet = "Tissue BLI log10",
           col_names = c("cell_line", "gene", "tissue", "p_original",
                         glue("ctrl_{1:14}"), glue("ko_{1:14}")),
           skip = 1) %>%
  drop_na(cell_line) %>%
  mutate(gene = toupper(gene),
         ctrl_1 = as.numeric(ctrl_1),
         ko_1 = as.numeric(ko_1),
         p_original = as.numeric(str_remove(p_original, "<"))) %>%
  pivot_longer(cols = matches("\\d")) %>%
  mutate(name = str_remove(name, "_\\d*"),
         name = factor(name, levels = c("ko", "ctrl"))) %>% 
  drop_na() %>%
  nest(data = c(name, value)) %>%
  mutate(t_test = map_dfr(data, ~t.test(value~name, data = .x) %>% tidy())) %>%
  unnest(t_test) %>%
  select(cell_line, gene, tissue, p_original, estimate, p_value = p.value,
         conf_low = conf.low, conf_high = conf.high) %>%
  mutate(p_value = p.adjust(p_value), .by = c(cell_line, gene)) %>%
  # ggplot(aes(x = p_value, y = p_original)) + geom_point() + geom_abline()
  mutate(gene = factor(gene,
                       levels = c("DHODH", "GART", "ASNS", "ASS1", 
                                  "PHGDH", "PYCR")),
         tissue = factor(tissue, levels = c("Brain", "Lungs", "Liver", "Kidney",
                                            "Bone", "Ovary"),
                         labels = c("Br", "Lu", "Li", "Ki", "Bo", "Ov")),
         cell_line = factor(cell_line,
                            levels = c("MDA-MB-231", "HCC1806","EO771")),
         significant = p_value < 0.05) %>% 
  ggplot(aes(x = tissue, y = estimate, color = cell_line, shape = significant,
             ymin = conf_low, ymax = conf_high)) +
  geom_hline(yintercept = 0, color = "gray", linewidth = 0.3) +
  geom_point(position = position_dodge(width = 0.6)) +
  geom_errorbar(width = 0, linewidth = 0.4,
                position = position_dodge(width = 0.6),
                show.legend = FALSE) +
  facet_wrap(~gene, nrow = 1) +
  labs(x = NULL, y = "Difference in\nmestastisis", color = NULL, shape = NULL) +
  scale_y_continuous(
    limits = c(-6.2, 2.6),
    breaks = c(-6, -4, -2, 0, 2),
    labels = glue("10<sup>{c(-6, -4, -2, 0, 2)}</sup>")
  ) +
  scale_color_manual(
    values = c("MDA-MB-231" = "#7570b3", HCC1806 = "#d95f02", EO771 = "#1b9e77")
  ) +
  scale_shape_manual(
    breaks = c(FALSE, TRUE),
    values = c(1, 19),
    labels = c("Not-significant", "Significant")
  ) +
  theme_classic() + 
  theme(
    legend.position = "bottom",
    axis.text.y = element_markdown(),
    axis.line = element_line(linewidth = 0.3),
    axis.ticks = element_line(linewidth = 0.3),
    strip.background = element_blank(),
    legend.box.spacing = unit(0, "pt"),
    legend.spacing.y = unit(-5, "pt"),
    legend.background = element_blank(),
    legend.box = "vertical",
    legend.key.size = unit(10, "pt"),
    legend.text = element_text(margin = margin(l = 3)))

ggsave("metastisis_dot_plot.png", width = 7.5, height = 2.5)  
```