---
layout: post
title: "Integrating curve fit data to improve the interpretability of dose response visualizations (CC420)"
blurb: "Going beyond the typical geom_smooth"
author: "PD Schloss"
date: 2026-04-29 12:00
comments: false
youtube: RZzxhqHbvSU
start_hash: 
end_hash: 
---

In this livestream, Pat fits a non-linear model through dose response data to recreate and improve upon a figure published in the journal Nature. He created the figure using R, readxl, dplyr, ggplot2, ggtext, patchwork, glue, and other tools from the tidyverse. The functions he used from these packages included aes, annotate, coord_cartesian, download.file, element_blank, element_line, element_markdown, element_text, ends_with, factor, filter, format, geom_errorbar, geom_hline, geom_jitter, geom_point, geom_segment, geom_smooth, geom_text, ggplot, ggsave, glue, guide_axis_logticks, if_else, labs, left_join, library, list, map, margin, mean, mean_sdl, mean_sdl, fun.args = , mutate, nest, nls, paste, pivot_longer, pivot_wider, plot_annotation, plot_layout, pnorm, read_excel, round, rowwise, scale_color_manual, scale_fill_manual, scale_shape_manual, scale_x_log10, scale_y_continuous, scale_y_log10, select, sqrt, stat_summary, theme, tidy, unit, and unnest. The original Nature Microbiology article can be [found here](https://www.nature.com/articles/s41586-026-10375-0). His critique of the visualization can be [seen here](https://youtu.be/n8Ze03_0Cmk). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```
library(tidyverse)
library(readxl)
library(glue)
library(broom)
library(ggtext)
library(patchwork)

url <- "https://static-content.springer.com/esm/art%3A10.1038%2Fs41586-026-10375-0/MediaObjects/41586_2026_10375_MOESM5_ESM.xlsx"
download.file(url, "figure2.xlsx")

d <- read_excel("figure2.xlsx", sheet = "Fig. 2b", skip = 3,
           col_names = c("bft", glue("wt_{1:3}"), glue("c4_{1:3}"),
                         glue("c3_{1:3}"), glue("gfp_{1:3}"))) %>%
  pivot_longer(-bft, names_to = c("genotype", "rep"), names_sep = "_",
               values_to = "ecadherin") %>%
  mutate(genotype = factor(genotype, levels = c("wt", "c4", "c3", "gfp")))

d %>%
  nest(data = -c(genotype, rep)) %>%
  mutate(fit = map(data,
                   ~nls(
                     ecadherin ~ Bottom + (Top-Bottom)/(1+(IC50/bft)^HillSlope),
                     start = list(Bottom = 0.15, Top = 1.0,
                                  IC50 = 0.03,
                                  HillSlope = -2),
                     data = .x  ) %>% tidy())) %>%
  unnest(fit) %>%
  filter(term == "IC50") %>%
  select(genotype, estimate, rep) %>%
  pivot_wider(names_from = "genotype", values_from = estimate) %>%
  mutate(wt_scaled = wt/mean(wt), c4_scaled = c4 / mean(wt),
         c3_scaled = c3 / mean(wt), gfp_scaled = gfp / mean(wt)) %>%
  select(rep, ends_with("scaled")) %>%
  pivot_longer(-rep, names_to = "genotype", values_to = "scaled_ec") %>%
  ggplot(aes(x = genotype, y = scaled_ec)) +
  stat_summary(fun = mean, geom = "col") + 
  geom_jitter() +
  scale_y_log10()


nls_fit <- d %>%
  nest(data = -genotype) %>%
  mutate(fit = map(data,
                   ~nls(
                     ecadherin ~ Bottom + (Top-Bottom)/(1+(IC50/bft)^HillSlope),
                     start = list(Bottom = 0.15, Top = 1.0,
                                  IC50 = 0.03,
                                  HillSlope = -2),
                     data = .x  ) %>% tidy())) %>%
  unnest(fit) %>%
  filter(term == "IC50") %>%
  select(genotype, estimate, std.error)

wt <- nls_fit %>%
  filter(genotype == "wt")

p_values <- nls_fit %>%
  filter(genotype != "wt") %>%
  rowwise() %>%
  mutate(z = (estimate - wt$estimate) / sqrt(std.error^2 + wt$std.error^2),
         p = pnorm(z, lower.tail = FALSE),
         pretty_p = if_else(p < 0.01,
                            "(P < 0.001)",
                            glue("(P = {round(p, 3)})"))) %>%
  select(genotype, pretty_p)

smoothed_plot <- d %>%
  ggplot(aes(x = bft, y = ecadherin, color = genotype, fill = genotype,
             shape = genotype)) +
  # geom_jitter(width = 0.1, alpha = 0.2) +
  geom_hline(yintercept = 0.5, color = "gray", linewidth = 0.2, 
             linetype = "dashed") +
  geom_hline(yintercept = 0, color = "black", linewidth = 0.2, 
             linetype = "solid") +
  stat_summary(geom = "point", size = 2, stroke = 0,
               fun.data = mean_sdl, fun.args = list(mult = 1)) +
  stat_summary(geom = "errorbar", width = 0.10, linewidth = 0.3,
               fun.data = mean_sdl, fun.args = list(mult = 1)) +
  geom_smooth(se = FALSE, linewidth = 0.3, 
              method = "nls",
              formula = y ~ Bottom + (Top-Bottom)/(1+(IC50/(10^x))^HillSlope),
              method.args = list(start = list(Bottom = 0.1, Top = 1.0,
                                              IC50 = 0.02,
                                              HillSlope = -2))) +
  # geom_segment(data = nls_fit,
  #              aes(x = estimate, y = 0, yend = 0.5, color = genotype)) +
  coord_cartesian(expand = FALSE, clip = "off") +
  scale_x_log10(
    limits = c(1e-3, 50),
    breaks = c(1e-3, 1e-2, 1e-1, 1e0, 1e1),
    labels = glue("10<sup>{-3:1}</sup>"),
    guide = guide_axis_logticks(long = 1, mid = 0.5, short = 0.5)
  ) +
  scale_y_continuous(
    limits = c(0, NA),
    breaks = c(0, 0.5, 1)) +
  scale_color_manual(
    values = c(wt = "black", c4 = "#FF2600", c3 = "#0433FF", gfp = "#942292"),
    labels = c(wt = "WT", c4 = "C4-KO", c3 = "C3-KO", gfp = "C4-KO::GFP-C4")
  ) +
  scale_fill_manual(
    values = c(wt = "black", c4 = "#FF2600", c3 = "#0433FF", gfp = "#942292"),
    labels = c(wt = "WT", c4 = "C4-KO", c3 = "C3-KO", gfp = "C4-KO::GFP-C4")
  ) +
  scale_shape_manual(
    values = c(wt = "circle filled", c4 = "square filled",
               c3 = "triangle filled", gfp = "triangle down filled"),
    labels = c(wt = "WT", c4 = "C4-KO", c3 = "C3-KO", gfp = "C4-KO::GFP-C4")
  ) +
  labs(
    shape = NULL,
    fill = NULL,
    color = NULL,
    x = "BFT concentration (ng ml<sup>-1</sup>)",
    y = "Normalized surface Ecad"
  ) +
  theme(
    axis.title.y = element_text(color = "black", size = 6.5),
    axis.text.y = element_text(color = "black", size = 6.5),
    axis.ticks.y = element_line(linewidth = 0.2),
    axis.line.y = element_line(linewidth = 0.2),
    
    legend.key.size = unit(8, "pt"),
    legend.key.width = unit(10, "pt"),
    legend.text = element_text(size = 6.5, margin = margin(l = 3)),
    legend.position = "inside",
    legend.position.inside = c(0.85, 0.8)
  )

ic50_plot <- nls_fit %>%
  left_join(p_values, by = "genotype") %>%
  mutate(ratio_to_wt = if_else(genotype == "wt",
                               "",
                               paste(format(round(estimate / first(estimate), 1), 1), pretty_p)),
         genotype = factor(genotype, levels = c("gfp", "c3", "c4", "wt"))) %>%
  ggplot(aes(x = estimate, y = genotype, xmin = estimate - 2 * std.error,
             xmax = estimate + 2 * std.error,
             color = genotype, shape = genotype, fill = genotype)) +
  geom_point(show.legend = FALSE, stroke = 0, size = 2) +
  geom_errorbar(show.legend = FALSE, 
                width = 1, linewidth = 0.3) +
  geom_text(aes(x = 30, label = ratio_to_wt), show.legend = FALSE,
            color = "black", size = 6, size.unit = "pt", hjust = 1) +
  scale_x_log10(
    expand = c(0, 0),
    limits = c(1e-3, 50),
    breaks = c(1e-3, 1e-2, 1e-1, 1e0, 1e1),
    labels = glue("10<sup>{-3:1}</sup>"),
    guide = guide_axis_logticks(long = 1, mid = 0.5, short = 0.5)) +
    scale_color_manual(
      values = c(wt = "black", c4 = "#FF2600", c3 = "#0433FF", gfp = "#942292"),
      labels = c(wt = "WT", c4 = "C4-KO", c3 = "C3-KO", gfp = "C4-KO::GFP-C4")
    ) +
    scale_fill_manual(
      values = c(wt = "black", c4 = "#FF2600", c3 = "#0433FF", gfp = "#942292"),
      labels = c(wt = "WT", c4 = "C4-KO", c3 = "C3-KO", gfp = "C4-KO::GFP-C4")
    ) +
    scale_shape_manual(
      values = c(wt = "circle filled", c4 = "square filled",
                 c3 = "triangle filled", gfp = "triangle down filled"),
      labels = c(wt = "WT", c4 = "C4-KO", c3 = "C3-KO", gfp = "C4-KO::GFP-C4")
    ) +
    labs(
      shape = NULL,
      fill = NULL,
      color = NULL,
      x = "BFT concentration (ng ml<sup>-1</sup>)",
      y = "IC<sub>50</sub>"
    ) +
  coord_cartesian(clip = "off") +
  annotate(geom = "text", x = 30, y = 4.25, label = "Relative to WT",
           hjust = 1, fontface = "bold",
           size = 6, size.unit = "pt") +
  theme(
    axis.text.y = element_blank(),
    axis.ticks.y = element_blank(),
    axis.title.y = element_markdown(color = "black", size = 6.5, angle = 0,
                                    margin = margin(r = -5), vjust = 0.5)
  )

smoothed_plot + theme(plot.margin = margin(0, 0, 6, 0)) +
  ic50_plot + theme(plot.margin = margin(0, 0, 0, 0)) +
  plot_layout(ncol = 1, heights = c(0.8, 0.2),
              axes = "collect") +
  plot_annotation(
    theme = theme(plot.margin = margin(3, 3, 3, 3))
  ) &
  theme(
    axis.title.x = element_markdown(color = "black", size = 6.5),
    axis.text.x = element_markdown(color = "black", size = 6.5),
    axis.ticks.x = element_line(linewidth = 0.2),
    axis.line.x = element_line(linewidth = 0.2),
    
    panel.grid = element_blank(),
    panel.background = element_blank(),
  )

ggsave("figure2bd.png", width = 3.5, height = 1.9)
```