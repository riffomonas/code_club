---
layout: post
title: "Recreating figures with a lot of data points with R's tidyverse (CC436)"
blurb: "Flipping a plot to see if it looks better"
author: "PD Schloss"
date: 2026-11-11 09:00
comments: false
youtube: zbJsviGJ6fM
start_hash: 
end_hash: 
---

The article discussed: https://www.nature.com/articles/s41586-026-10803-1
Code from this livestream: https://riffomonas.org/code_club/2026-11-11-distribution-livestream
Related critique: https://youtu.be/oRExT2tb1tI

Sign up for a complimentary consultation to help me learn your needs: https://calendly.com/pat-riffomonas/30min
My newsletter: https://shop.riffomonas.org/youtube 
Recorded workshops: https://riffomonas.org/workshops/
If you want to cite this video: https://journals.asm.org/doi/10.1128/mra.01310-22

* packages: tidyverse, readxl, ggtext, glue, scales

* functions: aes, annotate, arrange, as.numeric, col_lighter, coord_cartesian, download.file, element_blank, element_line, element_markdown, element_text, facet_wrap, factor, fct_reorder, format, geom_point, geom_text, geom_vline, ggplot, ggsave, glue, if_else, is.na, labs, library, median, mutate, n, paste, pivot_longer, position_dodge, read_excel, rename, round, scale_x_log10, stat_summary, str_replace_all, summarize, theme, theme_bw

This work was supported, in part, by a grant from the National Institutes of Health (R25GM166785)

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



```R
library(tidyverse)
library(readxl)
library(ggtext)
library(glue)
library(scales)

genome_size <- 2897.310462
pseudocount <- 0.5

download.file("https://static-content.springer.com/esm/art%3A10.1038%2Fs41586-026-10803-1/MediaObjects/41586_2026_10803_MOESM7_ESM.xlsx",
              "figure_2.xlsx")

d <- read_excel("figure_2.xlsx", range = "A2:G1224") %>%
  pivot_longer(cols = c("SBS", "DBS", "ID", "SV", "CN"),
               names_to = "mutation", values_to = "count") %>%
  rename(samples = Samples,
         state_therapy = `Therapy-State`) %>%
  mutate(
    count = if_else(is.na(count) | count == 0, pseudocount, count),
    rate = count / genome_size) %>%
  arrange(state_therapy, mutation, rate) %>%
  mutate(index = 1:n(),
         n = n(),
         .by = c(state_therapy, mutation)) %>%
  mutate(
     mutation = factor(mutation,
                       levels = c("SBS", "ID", "SV", "CN", "DBS"),
                       labels = c("Single-base\nsubstitutions",
                                  "Insertions and\ndeletions",
                                  "Structural\nvariants",
                                  "Copy number\nalterations",
                                  "Double-base\nsubstitutions")),
    state_therapy = str_replace_all(state_therapy, "[ \r]\n?", "\n"),
    state_therapy = factor(state_therapy,
                           levels = c("Advanced\nTreated", "Advanced\nNaive",
                                      "Primary\nTreated", "Primary\nNaive",
                                      "All\nSamples")),
    pretty_state_therapy = glue("{state_therapy} ({n})"),
    pretty_state_therapy = fct_reorder(pretty_state_therapy,
                                       as.numeric(state_therapy)))


summary_data <- d %>%
  summarize(
    median = median(rate),
    .by = c(pretty_state_therapy, mutation)
  ) %>%
  mutate(
    pretty_median = format(round(median, 3), nsmall = 3),
    x = median * 1.5
    )

d %>%
  ggplot(aes(x = rate, y = pretty_state_therapy)) +
  annotate(
    geom = "rect",
    xmin = 0.5 * pseudocount / genome_size,
    xmax = 1e3,
    ymin = 3.5, ymax = 4.5, fill = col_lighter("#A0B9D5", 15)
  )  +
  annotate(
    geom = "rect",
    xmin = 0.5 * pseudocount / genome_size,
    xmax = 1e3,
    ymin = 0.5, ymax = 1.5, fill = col_lighter("#F3D9D5", 5)
  ) +
  stat_summary(aes(group = paste(pretty_state_therapy, mutation)),
               geom = "crossbar", fun = median,
               linewidth = 0.3, lineend = "round", width = 0.75) +
  geom_text(data = summary_data,
            aes(x = x, y = as.numeric(pretty_state_therapy) - 0.25, label = pretty_median),
            hjust = 0, size = 8, size.unit = "pt") +
  geom_point(aes(group = index), size = 0.1, color = "gray60", 
             position = position_dodge(width = 0.75)) +
  geom_vline(xintercept = 1.5 * pseudocount / genome_size,
             color = "gray80", linewidth = 0.3, linetype = "dashed") +
  scale_x_log10(
    breaks = 10^(-3:3),
    labels = glue("10<sup>{-3:3}</sup>")
  ) +
  coord_cartesian(expand = FALSE, ylim = c(0.5, 5.5)) +
  facet_wrap(~mutation, nrow = 1) +
  labs(x = "Somatic mutations (Mb<sup>-1</sup>)") +
  theme_bw() +
  theme(
    panel.grid = element_blank(),
    strip.background = element_blank(),
    
    axis.title.y = element_blank(),
    axis.text.y = element_text(hjust = 0, size = 7),
    axis.ticks.y = element_blank(),
    
    axis.title.x = element_markdown(size = 8),
    axis.text.x = element_markdown(size = 6),
    axis.ticks.x = element_line(linewidth = 0.2)
  )

ggsave("figure_2a.png", width = 7.500, height = 2.462)
```