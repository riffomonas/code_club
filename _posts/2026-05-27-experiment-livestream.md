---
layout: post
title: "Converting a stacked bar plot to something better (CC427)"
blurb: "No more stacked bar plots!"
author: "PD Schloss"
date: 2026-05-27 10:00
comments: false
youtube: Q9f7JQhl644
start_hash: 
end_hash: 
---

* The article discussed: https://www.nature.com/articles/s41564-026-02350-2
* Related critique: https://youtu.be/g8wdteeviao
* Code from this livestream:
* Sign up for a complimentary consultation to help me learn your needs: https://calendly.com/pat-riffomonas/30min
* My newsletter: https://shop.riffomonas.org/youtube 
* Recorded workshops: https://riffomonas.org/workshops/
* If you want to cite this video: https://journals.asm.org/doi/10.1128/mra.01310-22


* packages: tidyverse, readxl, ggplot2, dplyr, purrr, broom

* functions: aes, annotate, as.numeric, case_when, coord_cartesian, distinct, download.file, element_blank, element_line, element_rect, element_text, everything, facet_wrap, factor, filter, geom_line, geom_polygon, geom_segment, geom_text, ggplot, ggsave, if_else, inner_join, kruskal.test, labs, length, levels, library, map, map_dbl, margin, max, median, mutate, nest, p.adjust, pairwise.wilcox.test, paste, pivot_longer, pivot_wider, read_excel, rep, scale_color_identity, scale_x_continuous, scale_y_continuous, select, separate_wider_delim, seq, theme, theme_classic, tidy, unit, unnest, wilcox.test


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```
library(tidyverse)
library(readxl)
library(broom)

download.file("https://static-content.springer.com/esm/art%3A10.1038%2Fs41564-026-02350-2/MediaObjects/41564_2026_2350_MOESM6_ESM.xlsx",
              "fig_3.xlsx")

taxa <- c("Akkermansiaceae", "Atopobiaceae", "Bacteroidaceae",
          "Bifidobacteriaceae", "Desulfovibrionaceae", "Enterobacteriaceae",
          "Erysipelatoclostridiaceae", "Erysipelotrichaceae", "Lachnospiraceae",
          "Lactobacillaceae", "Muribaculaceae", "Oscillospiraceae",
          "Peptostreptococcaceae", "Prevotellaceae", "Rikenellaceae", 
          "Ruminococcaceae", "Streptococcaceae", "Sutterellaceae",
          "Tannerellaceae", "Uncultured", "Others")

mouse <- rep(1:8, length(taxa))
col_names <- c("treatment", 
               paste(rep(taxa, each = 8), mouse, sep = "_"))

dodge <- 0.2

d <- read_excel("fig_3.xlsx", sheet = "Fig. 3f", range = "C4:FO11",
           col_names = col_names) %>%
  mutate(time = rep(c("Before", "After"), each = 4), .before = everything()) %>%
  pivot_longer(cols = -c(time, treatment),
               names_to = "family_mouse", values_to = "rel_abund") %>%
  separate_wider_delim(family_mouse, delim = "_", 
                       names = c("family", "mouse")) %>%
  mutate(treatment = factor(treatment, 
                            levels = c("Ctrl", "Acr", "Abx", "Abx+Acr"),
                            labels = c("Ctrl", "Acr", "Abx", "Abx+\nAcr")))

### pairwise comparisons

p <- d %>%
  pivot_wider(names_from = time, values_from = rel_abund) %>%
  nest(data = -c(treatment, family)) %>%
  mutate(test = map(.x = data,
                    ~wilcox.test(.x$Before, .x$After, paired = TRUE) %>%
                      tidy()),
         diff = map_dbl(.x = data, ~median(.x$After - .x$Before))) %>%
  unnest(test) %>%
  mutate(p.value = p.adjust(p.value, method = "BH"), .by = treatment) %>%
  mutate(color = case_when(p.value < 0.05 & diff < 0 ~ "blue",
                           p.value < 0.05 & diff > 0 ~ "red",
                           TRUE ~ "gray")) %>%
  select(treatment, family, color)
  
### difference of differences

dd <- d %>%
  pivot_wider(names_from = time, values_from = rel_abund) %>%
  mutate(diff = After - Before) %>%
  select(treatment, family, diff, After, Before) %>%
  nest(data = -family) %>%
  mutate(test = map(data, ~kruskal.test(.x$diff, g = .x$treatment) %>%
                      tidy()),
         max = map_dbl(data, ~max(c(.x$After, .x$Before)))) %>%
  unnest(test) %>%
  mutate(p.kruskal = p.adjust(p.value, method = "BH")) %>%
  select(family, data, p.kruskal, max) %>%
  mutate(test = map(data, ~pairwise.wilcox.test(.x$diff, .x$treatment,
                                                p.adjust.method = "BH") %>%
                      tidy())) %>%
  unnest(test) %>%
  select(family, p.kruskal, group1, group2, p.pw = p.value, max) %>%
  mutate(group1 = factor(group1,
                         levels = c("Ctrl", "Acr", "Abx", "Abx+\nAcr")) %>%
           as.numeric(),
         group2 = factor(group2,
                         levels = c("Ctrl", "Acr", "Abx", "Abx+\nAcr")) %>%
           as.numeric(),
         y = case_when(group1 == 4 & group2 == 2 ~ max + 0.09,
                       group1 == 3 & group2 == 2 ~ max + 0.06,
                       group1 == 4 & group2 == 3 ~ max + 0.03,
                       TRUE ~ 0)) %>%
  filter(group1 != 1 & group2 != 1) %>%
  filter(p.kruskal < 0.05 & p.pw < 0.05)




d %>%
  inner_join(., p, by = c("treatment", "family")) %>%
  filter(treatment != "Ctrl") %>%
  filter(family != "Others") %>%
  mutate(x = as.numeric(treatment),
         x = if_else(time == "Before", x - dodge, x + dodge)) %>%
  # filter(family == "Bacteroidaceae") %>%
  ggplot(aes(x = x, y = rel_abund, color = color,
             group = paste(mouse, treatment))) +
  geom_line(linewidth = 0.3) +
  geom_text(data = d %>% distinct(family) %>% filter(family != "Others"), 
            mapping = aes(label = family),
            x = 3, y = 0.76, color = "black", size = 6, size.unit = "pt",
            inherit.aes = FALSE) +
  geom_segment(data = filter(dd, family != "Others"),
               mapping = aes(x = group1, xend = group2, y = y),
               linewidth = 0.3,
               inherit.aes = FALSE) +
  facet_wrap(~family, ncol = 5) +
  labs(x = NULL, y = "Relative abundance (%)") +
  scale_color_identity() +
  scale_x_continuous(
    breaks = 2:4,
    labels = levels(d$treatment)[-1]
  ) +
  scale_y_continuous(
    breaks = seq(0, 0.6, 0.2),
    labels = seq(0, 60, 20)
  ) +
  coord_cartesian(ylim = c(-0.02, 0.8), xlim = c(1.5, 4.5),
                  expand = FALSE, clip = "off") +
  theme_classic() +
  theme(
    axis.text = element_text(size = 6),
    axis.text.y = element_text(margin = margin(r = 1)),
    axis.title = element_text(size = 7),
    
    axis.line = element_blank(),
    axis.ticks = element_line(linewidth = 0.2),
    axis.ticks.length.y = unit(2, "pt"),
    
    strip.text = element_blank(),
    strip.background = element_blank(),
    
    panel.background = element_rect(color = "black", linewidth = 0.2),
    panel.spacing = unit(1, "pt"),
    
    plot.margin = margin(2, 1, 2, 1)
  )# +
  # annotate(layout = 3,
  #          geom = "polygon",
  #          x = c(1.5, 4.5, 4.5, 1.5),
  #          y = c(-0.02, -0.02, 0.8, 0.8), color = "red", fill = NA)


ggsave("fig_3f_refactor.png", width = 5, height = 4)  
```