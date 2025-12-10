---
layout: post
title: "Remaking a jittered plot with P values from Nature Microbiology in R with ggplot2 (CC389)"
blurb: "Someone listened to me!"
author: "PD Schloss"
date: 2025-12-10 12:00
comments: false
youtube: nMkSgEXv3tc
start_hash: 
end_hash: 
---

Pat recreates a paired of jittered plots that have annotations for the median, P-values, and grupings. The original panels were included as part of an article published in Nature Microbiology. He created the figure using R, dplyr, ggplot2, ggtext, readxl, patchwork, and other tools from the tidyverse. The functions he used from these packages included aes, annotate, bind_rows, c, coord_cartesian, download.file, drop_na, element_line, element_markdown, element_text, factor, filter, font_add_google, function, geom_hline, geom_jitter, geom_richtext, ggplot, ggsave, glue, if, if_else, labs, library, map2, margin, mutate, n, nest, pivot_longer, pivot_wider, plot_annotation, pluck, position_jitter, read_excel, rep, scale_color_manual, scale_fill_manual, scale_x_discrete, scale_y_log10, select, separate_wider_delim, showtext_auto, showtext_opts, stat_summary, theme, theme_classic, tibble, unique, unit, unlist, unnest, and wilcox.test. The newsletter describing this visualization at a 30,000 ft view can be [found here](https://shop.riffomonas.org/posts/adding-additional-layers-of-text-and-titles-to-an-x-axis-with-ggplot2-in-r). You can find the original article presenting the figure at [*Nature Microbiology*](https://www.nature.com/articles/s41564-025-02162-w). Here's [a video critiquing](https://riffomonas.org/code_club/2025-12-08-invader-displacement-critique) the original figure. If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```R
library(tidyverse)
library(readxl)
library(ggtext)
library(glue)
library(showtext)
library(patchwork)

font_add_google("Nunito", "nunito")
showtext_opts(dpi = 300)
showtext_auto()

xlsx_link <- "https://static-content.springer.com/esm/art%3A10.1038%2Fs41564-025-02162-w/MediaObjects/41564_2025_2162_MOESM4_ESM.xlsx"
xlsx_file <- "figure2_data.xlsx"

download.file(xlsx_link, xlsx_file)

panel_f_g_data <- bind_rows(
    "Invasion" = read_excel(xlsx_file, range = "B39:G48"),
    "Displacement" = read_excel(xlsx_file, range = "B52:G61"),
    .id = "panel") %>%
  mutate(replicate = 1:n()) %>%
  pivot_longer(-c(panel, replicate), names_to = "condition",
               values_to = "density") %>%
  drop_na() %>%
  separate_wider_delim(cols = condition, cols_remove = FALSE,
                       delim = ", ",
                       names = c("genotype", "toxin")) %>%
  mutate(fill_variable = if_else(panel == "Invasion",
                                 toxin,
                                 genotype))


p_values <- panel_f_g_data %>%
  select(panel, genotype, toxin, density) %>%
  nest(data = density) %>%
  pivot_wider(values_from = data, names_from = toxin) %>%
  mutate(p_e2 = map2(`-`, colE2,
                     ~wilcox.test(x = unlist(.x), y = unlist(.y),
                                  exact = FALSE) %>%
                       pluck("p.value")),
         p_k = map2(`-`, colK,
                     ~wilcox.test(x = unlist(.x), y = unlist(.y),
                                  exact = FALSE) %>%
                      pluck("p.value"))) %>%
  unnest(c(p_e2, p_k)) %>%
  select(panel, genotype, colE2 = p_e2, colK = p_k) %>%
  pivot_longer(c(colE2, colK), names_to = "toxin", values_to = "p") %>%
  mutate(x = c(1.5, 2, 4.5, 5, 1.5, 2, 4.5, 5),
         y = c(1e6, 1e7, 1e10, 1e11, 1e10, 1e11, 1e10, 1e11),
         pretty_p = glue("*P* = {signif(p, 2)}"))


plot_panels_f_g <- function(p_title){
  
  y_axis_tag <- if_else(p_title == "Invasion", "invader", "resident")
  
  sig_line_y <- c(4e5, 4e6)
  
  if(p_title == "Displacement"){
    sig_line_y <- c(4e9, 4e10)
  }
  
  panel_f_g_data %>%
    filter(panel == p_title) %>%
    mutate(condition = factor(condition, levels = unique(condition))) %>%
    ggplot(aes(x = condition, y = density, color = panel,
               fill = fill_variable)) +
    geom_hline(yintercept = 200, linewidth = 0.2, color = "gray") +
    geom_jitter(shape = 21, show.legend = FALSE, 
                position = position_jitter(width = 0.3, seed = 19760620)) +
    stat_summary(fun = median, geom = "crossbar", show.legend = FALSE,
                 color = "black", linewidth = 0.1, width = 0.8) +
    labs(title = p_title,
         x = NULL,
         y = glue("End-point density of<br>{y_axis_tag} (CFU ml<sup>-1</sup>)")
     ) +
    scale_fill_manual(
      values = c("-" = "white", "colE2" = "black", "colK" = "#616161",
                 "WT" = "white", "srl" = "gray")
    ) +
    scale_color_manual(
      values = c("Invasion" = "#E60000", "Displacement" = "#6298D0")
    ) +
    scale_y_log10(
      breaks = c(10^(seq(1, 9, 2))),
      labels = glue("10<sup>{seq(1, 9, 2)}</sup>")
    ) +
    scale_x_discrete(
      labels = rep(c("\u2014", "E2", "K"), 2)
    ) +
    geom_richtext(data = tibble(x = c(2, 5), y = 0.05, label = c("WT", "&Delta;*srlAEB*")),
                  aes(x = x, y = y, label = label), inherit.aes = FALSE,
                  family = "nunito", size = 2.5, fill = NA, label.color = NA,
                  label.padding = unit(0, "pt")) +
    annotate(geom = "text",
             x = 0.5, y = c(1, 0.05), label = c("Invader toxin", "Resident"),
             family = "nunito", size = 7, size.unit = "pt", hjust = 1) +
    annotate(geom = "segment", linewidth = 0.2,
             x = c(0.6, 3.6), xend = c(3.4, 6.4),
             y = 0.3) +
    annotate(geom = "segment", linewidth = 0.2,
             x = c(0.6, 3.4, 3.6, 6.4),
             y = 0.3, yend = 0.5) +
    geom_richtext(data = p_values %>% filter(panel == p_title),
                  aes(x = x, y = y, label = pretty_p), inherit.aes = FALSE,
                  family = "nunito", size = 2, fill = NA, label.color = NA,
                  label.padding = unit(0, "pt")) +
    annotate(geom = "segment", linewidth = 0.2,
             x = c(1, 1, 4, 4), xend = c(2, 3, 5, 6),
             y = c(sig_line_y, 4e9, 4e10)) +
    annotate(geom = "segment", linewidth = 0.2,
             x = c(1, 2, 1, 3, 4, 5, 4, 6),
             y = c(rep(sig_line_y, each = 2), 4e9, 4e9, 4e10, 4e10),
             yend = c(rep(sig_line_y/2, each = 2), 2e9, 2e9, 2e10, 2e10)) +
    coord_cartesian(clip = "off", expand = FALSE,
                    xlim = c(0.5, 6.5),
                    ylim = c(1e1, 1e9)) +
    theme_classic() + 
    theme(
      text = element_text(family = "nunito"),
      plot.title = element_text(hjust = 0.5, size = 7, margin = margin(b = 25)),
      plot.margin = margin(t = 3, r = 8, b = 15, l = 15),
      axis.text.y = element_markdown(size = 6),
      axis.text.x = element_markdown(size = 7),
      axis.title.y = element_markdown(size = 7),
      axis.line = element_line(linewidth = 0.2),
      axis.ticks = element_line(linewidth = 0.2),
      axis.ticks.length = unit(3, "pt"),
    )
}

plot_panels_f_g("Invasion") + plot_panels_f_g("Displacement") +
  plot_annotation(
    theme = theme(plot.margin = margin(0, 0, 0, 0))
    )

ggsave("panels_f_g.png", width = 7, height = 1.9)
```