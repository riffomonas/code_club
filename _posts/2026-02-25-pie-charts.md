---
layout: post
title: "Refactoring a pie chart with ggplot2 in R to make interpretation easier for readers (CC402)"
blurb: "Stacked barplots aren't always bad"
author: "PD Schloss"
date: 2026-02-25 12:00
comments: false
youtube: GoCg9xutHsM
start_hash: 
end_hash: 
---

In this livestream, Pat refactors a set of pie charts from a paper recently published in Nature Microbiology. The figure panel consists of 8 pie charts showing the response of women with recurrent cystitis to an immunologic and an antibiotic. The goal was to restructure the pie charts to make them easier for the audience to interpret. Pat recreated the original pie charts and refactored them into a stacked bar plot and dotplot. He created the figures using R, dplyr, ggplot2, ggh4x, ggtext, readxl, DescTools, and other tools from the tidyverse. The functions he used from these packages included CochranArmitageTest, aes, arrange, case_when, column_to_rownames, coord_radial, count, download.file, drop_na, element_blank, element_text, facet_wrap, facet_wrap2, factor, filter, format, geom_col, geom_point, geom_text, ggplot, ggsave, if_else, inner_join, labs, library, list, map, margin, mutate, nest, paste0, pivot_longer, pivot_wider, position_dodge, position_stack, read_excel, rename_all, round, scale_fill_manual, scale_x_continuous, scale_y_discrete, select, seq, str_replace, str_to_title, strip_split, sum, theme, tidy, tolower, unique, unit, and unnest. The original Nature Microbiology article can be [found here](https://www.nature.com/articles/s41564-026-02262-1). My critique of the original figure can be [found here](https://youtu.be/zWptjqCr4DY). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```R
library(tidyverse)
library(readxl)
library(ggh4x)
library(DescTools)
library(broom)

data_url <- "https://static-content.springer.com/esm/art%3A10.1038%2Fs41564-026-02262-1/MediaObjects/41564_2026_2262_MOESM5_ESM.xlsx"

download.file(data_url, "cystitis_data.xlsx")

d <- read_excel("cystitis_data.xlsx", sheet = "Fig2") %>%
  rename_all(tolower) %>%
  select(patient, treatment, visit, dynamics) %>%
  drop_na(dynamics) %>%
  mutate(treatment = str_to_title(treatment),
         response = case_when(dynamics <= 1 ~ "better",
                              dynamics == 2 ~ "some",
                              dynamics >= 3 ~ "worse"),
         visit = if_else(visit == "6months",
                         "6 months",
                         str_replace(visit, "Day(\\d*)", "\\1 days")),
         visit = factor(visit, 
                        levels = c("5 days", "30 days", 
                                   "15 days", "6 months"))) %>%
  count(treatment, visit, response) %>%
  pivot_wider(names_from = response, values_from = n, values_fill = 0) %>%
  pivot_longer(cols = c(better, some, worse),
               names_to = "response", values_to = "n") %>%
  mutate(fraction = n / sum(n), .by = c(treatment, visit),
         response = factor(response, levels = c("better", "some", "worse")))


ca_test <- d %>%
  select(-fraction) %>%
  nest(data = -visit) %>%
  mutate(test = map(data,
                    ~pivot_wider(.x, names_from = "response", values_from = n) %>%
                      column_to_rownames("treatment") %>%
                      CochranArmitageTest() %>%
                      tidy())) %>%
  unnest(test) %>%
  select(visit, p.value) %>%
  mutate(
    pretty_p = paste0("P = ", format(round(p.value, digits = 3), nsmall = 3)),
    response = "some",
    treatment = "Anakinra")
  
# Pie chart code ----

d %>%
  filter(n != 0) %>%
  ggplot(aes(x = 1, y = fraction, fill = response, label = n)) +
  geom_col(color = "white", linewidth = 0.2) +
  geom_text(position = position_stack(vjust = 0.5),
            size = 7, size.unit = "pt") +
  geom_text(data = ca_test, aes(x = 1.6, y = 0.65, label = pretty_p),
            hjust = 0, vjust = 0.5, size = 7, size.unit = "pt") +
  coord_radial(theta = "y", expand = FALSE, reverse = "theta", clip = "off") +
  facet_wrap2(.~visit + treatment, nrow = 2,
              strip = strip_split(position = c("left", "top"),
                                  text_x = list(
                                    element_text(margin = margin(b = -5)),
                                    element_text(margin = margin(b = -5)),
                                    element_text(margin = margin(b = -5)),
                                    element_text(margin = margin(b = -5)),
                                    element_blank(), element_blank(),
                                    element_blank(), element_blank()),
                                  background_x = element_blank(),
                                  text_y = element_text(angle = 0, hjust = 1,
                                                        margin = margin(l = 10,
                                                                        r = 0)),
                                  background_y = element_blank(),
                                  clip = "off")) +
  labs(
    x = NULL, y = NULL, fill = NULL
  ) +
  scale_fill_manual(
    values = c(better = "#63BE7B", some = "#FDC37C", worse = "#F76A67"),
    labels = c(better = "All or some symptoms are gone (0-1)",
               some = "Some symptoms are gone (2)",
               worse = "Same symptoms or worse (3-4)")) +
  theme(
    plot.margin = margin(),
    panel.background = element_blank(),
    panel.spacing.x = unit(0, "pt"),
    panel.spacing.y = unit(-20, "pt"),
    panel.grid = element_blank(), #element_line(color = "black"),
    legend.position = "bottom",
    legend.key.size = unit(8, "pt"),
    legend.text = element_text(size = 7, margin = margin(l = 2, r = 5)),
    legend.box.spacing = unit(0, "pt"),
    legend.margin = margin(),
    axis.text = element_blank(),
    axis.ticks = element_blank()
  )

ggsave("pie-charts.png", width = 6, height = 2.8)


# Stacked bar chart code ----

ca_test %>%
  select(visit, pretty_p) %>%
  inner_join(., d, by = "visit") %>%
  mutate(
    visit = factor(visit, levels = c("5 days", "15 days", 
                                     "30 days", "6 months"))
  ) %>%
  arrange(visit) %>%
  mutate(
    visit_p = paste0(visit, "\n(", pretty_p, ")"),
         visit_p = factor(visit_p, levels = unique(visit_p))) %>%
  filter(n != 0) %>%
  ggplot(aes(y = treatment, x = fraction, fill = response, label = n)) +
  geom_col(color = "white", linewidth = 0.2,
           position = position_stack(reverse = TRUE)) +
  geom_text(position = position_stack(vjust = 0.5, reverse = TRUE),
            size = 7, size.unit = "pt") +
  facet_wrap(~visit_p, ncol = 1, strip.position = "left") +
  labs(
    x = NULL, y = NULL, fill = NULL
    ) +
  scale_x_continuous(
    expand = c(0, 0),
    breaks = seq(0, 1, 0.25),
    labels = seq(0, 100, 25)) +
  scale_y_discrete(position = "right", expand = c(0, 0)) +
  scale_fill_manual(
    values = c(better = "#63BE7B", some = "#FDC37C", worse = "#F76A67"),
    labels = c(better = "All or some symptoms are gone (0-1)",
               some = "Some symptoms are gone (2)",
               worse = "Same symptoms or worse (3-4)")) +
  theme(
    # plot.margin = margin(),
    panel.background = element_blank(),
    panel.grid = element_blank(),
    legend.position = "bottom",
    legend.key.size = unit(8, "pt"),
    legend.text = element_text(size = 7, margin = margin(l = 2, r = 5)),
    legend.box.spacing = unit(2, "pt"),
    legend.margin = margin(),
    strip.text.y.left = element_text(angle = 0),
    strip.background = element_blank(),
    axis.text.y.right = element_text(size = 7, color = "black"),
    axis.text.x = element_text(color = "black", size = 8),
    axis.ticks = element_line(linewidth = 0.2),
    axis.ticks.y.right = element_blank()
  )

ggsave("stacked-barplot.png", width = 6, height = 2.8)


# Dot plot ----

ca_test %>%
  select(visit, pretty_p) %>%
  inner_join(., d, by = "visit") %>%
  mutate(
    visit = factor(visit, levels = c("5 days", "15 days", 
                                     "30 days", "6 months"))
  ) %>%
  arrange(visit) %>%
  mutate(
    visit_p = paste0(visit, "\n(", pretty_p, ")"),
    visit_p = factor(visit_p, levels = unique(visit_p))) %>%
  # filter(n != 0) %>%
  ggplot(aes(y = treatment, x = fraction, fill = response)) +
  geom_point(shape = 21, position = position_dodge(width = 0.5)) +
  facet_wrap(~visit_p, ncol = 1, strip.position = "left") +
  labs(
    x = NULL, y = NULL, fill = NULL
  ) +
  scale_x_continuous(
    expand = expansion(add = c(0.0, 0.02)),
    breaks = seq(0, 1, 0.25),
    labels = seq(0, 100, 25)) +
  scale_y_discrete(position = "right") +#, expand = c(0, 0)) +
  coord_cartesian(clip = "off") +
  scale_fill_manual(
    values = c(better = "#63BE7B", some = "#FDC37C", worse = "#F76A67"),
    labels = c(better = "All or some symptoms are gone (0-1)",
               some = "Some symptoms are gone (2)",
               worse = "Same symptoms or worse (3-4)")) +
  theme(
    panel.background = element_blank(),
    panel.grid.major.x = element_blank(),
    panel.grid.major.y = element_line(linewidth = 0.2, color = "darkgray"),
    panel.grid.minor = element_blank(),
    legend.position = "bottom",
    legend.key.size = unit(8, "pt"),
    legend.text = element_text(size = 7, margin = margin(l = 2, r = 5)),
    legend.box.spacing = unit(2, "pt"),
    legend.margin = margin(),
    strip.text.y.left = element_text(angle = 0),
    strip.background = element_blank(),
    axis.text.y.right = element_text(size = 7, color = "black"),
    axis.text.x = element_text(color = "black", size = 8),
    axis.ticks = element_line(linewidth = 0.2),
    axis.ticks.y.right = element_blank()
  )

ggsave("dotplot.png", width = 6, height = 2.8)
```