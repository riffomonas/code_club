---
layout: post
title: "Refactoring a collection of panels to make design more cohesive and improve interpretability (CC404)"
blurb: "Fun with facets and statistics!"
author: "PD Schloss"
date: 2026-03-03 12:00
comments: false
youtube: JVTiYFCAu4o
start_hash: 
end_hash: 
---

In this livestream, Pat refactors a set of plots recently published in Nature into a single faceted figure with statistical test results embedded into each panel. The goal was to restructure the figures to make them more cohesive so that they are easier for the audience to interpret. He created the figures using R, dplyr, ggplot2, ggtext, readxl, emmeans, multcomp, glue, broom, and other tools from the tidyverse. The functions he used from these packages included aes, as.numeric, cld, cor.test, element_blank, element_line, element_markdown, element_text, emtrends, facet_grid, factor, filter, format, geom_jitter, geom_richtext, geom_smooth, ggplot, ggsave, glue, if_else, inner_join, labeller, labs, library, lm, map, margin, mutate, nest, p.adjust, pivot_wider, position_jitter, read_excel, round, scale_color_manual, scale_shape_manual, scale_x_continuous, scale_y_continuous, select, seq, str_remove_all, summary, theme, tidy, unit, and unnest. The original Nature article [can be found here](https://www.nature.com/articles/s41586-025-10039-5). A video critiquing the original version of the figure [can be found here](https://youtu.be/LCOSJqlRAXk). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```R
library(multcomp)
library(tidyverse)
library(readxl)
library(ggtext)
library(broom)
library(glue)
library(emmeans)


# Get and format data ----

# Get data from figshare:
# https://figshare.com/articles/dataset/Rising_atmospheric_CO_sub_2_sub_reduces_nitrogen_availability_in_boreal_forests_dataset_/30675002?file=59748374
# Save xlsx file to directory

d <- read_excel("2024-12-26604.xlsx", sheet = "Rev2") %>%
  select(species = Species,
         year = AvgYr,
         region = Regions,
         d15N) %>%
  mutate(year = as.numeric(year), 
         region = factor(region,
                         levels = c("NOSE", "CESE", "SESE", "SWSE")))

# Run statistics ----

# d %>%
#   filter(species == "spruce" & region == "SWSE") %>%
#   cor.test(~year + d15N, data = .) %>%
#   tidy()

# d %>%
#   filter(species == "spruce" & region == "SWSE") %>%
#   lm(d15N ~ year, data = .) %>% 
#   summary() %>%
#   tidy()


grouping <- d %>%
  nest(data = -species) %>%
  mutate(multi_linear = map(data, ~lm(d15N ~ year * region, data = .x) %>%
                              emtrends(spec = "region", var = "year") %>%
                              cld(Letters = letters, decreasing = TRUE) %>%
                              tidy() %>%
                              mutate(.group = str_remove_all(.group, " ")) %>%
                              select(region, group = .group))) %>%
  unnest(multi_linear) %>%
  mutate(region = factor(region, levels = c("NOSE", "CESE", "SESE", "SWSE"))) %>%
  select(species, region, group)

stats <- d %>%
  nest(nitrogen = c(year, d15N)) %>%
  mutate(correlation = map(nitrogen,
                           ~cor.test(~year + d15N, data = .x) %>%
                             tidy() %>%
                             mutate(r2 = estimate^2,
                                    cor.p = p.adjust(p.value)) %>%
                             select(r2, cor.p)),
         model = map(nitrogen,
                     ~lm(d15N ~ year, data = .x) %>%
                       tidy() %>%
                       select(term, estimate, p.value) %>%
                       mutate(term = str_remove_all(term, "[()]")) %>%
                       pivot_wider(names_from = "term",
                                   values_from = c(estimate, p.value)) %>%
                       select(intercept = estimate_Intercept,
                              slope = estimate_year,
                              slope.p = p.value_year) %>%
                       mutate(slope.p = p.adjust(slope.p)))) %>%
  unnest(c(correlation, model)) %>%
  select(-nitrogen, -cor.p) %>%
  inner_join(., grouping, by = c("species", "region")) %>%
  mutate(
    slope = round(slope, digits = 2) %>% format(nsmall = 2),
    intercept = round(intercept, digits = 0),
    r2 = round(r2, digits = 3) %>% format(nsmall = 3),
    slope.p = if_else(slope.p < 0.001, "*P* < 0.001",
                      glue("*P* = {round(slope.p, digits = 3) %>% format(nsmall = 3)}")),
    pretty_stats = glue("*y* = {slope} *x* + {intercept} (**{group}**)<br>
                        *R<sup>2</sup>* = {r2}<br>
                        {slope.p}"))


  
# Plot data and statistics ----

pretty_regions = c(NOSE = "North", CESE = "Central",
                   SESE = "Southeast", SWSE = "Southwest")

pretty_species = c(pine = "*P. sylvestris*",
                   spruce = "*P. abies*")

d %>%
  ggplot(aes(x = year, y = d15N, color = region, shape = species)) +
  geom_jitter(alpha = 0.5, size = 1.2, show.legend = FALSE,
              position = position_jitter(width = 2, height = 0,
                                         seed = 19760620)) +
  geom_smooth(method = "lm", formula = "y ~ x", color = "black",
              se = FALSE, linewidth = 0.3) +
  geom_richtext(data = stats,
                mapping = aes(x = 2015, y = 11,
                              label = pretty_stats),
                color = "black", hjust = 1, vjust = 1, size = 2.3,
                label.padding = unit(0, "pt"), label.colour = NA, fill = NA) +
  facet_grid(region ~ species, axes = "all", axis.labels = "margins",
             labeller = labeller(region = pretty_regions,
                                 species = pretty_species)) +
  scale_color_manual(
    values = c(NOSE = "#B9E4BD",
               CESE = "#7BCCC5",
               SESE = "#43A3CB",
               SWSE = "#0767AC")
  ) +
  scale_shape_manual(
    values = c(pine = 19, spruce = 17)
  ) +
  scale_y_continuous(limits = c(-11, 11), expand = c(0, 0)) +
  scale_x_continuous(limits = c(1947, 2016), expand = c(0, 0),
                     breaks = seq(1950, 2010, 10)) +
  labs(
    x = "Year",
    y = "Wood isotope ratio (&delta;<sup>15</sup>N[&permil;])"
  ) +
  theme(
    panel.grid = element_blank(),
    panel.background = element_blank(),
    axis.line.x = element_line(linewidth = 0.2, color = "black"),
    axis.line.y = element_line(linewidth = 0.2, color = "black"),
    axis.ticks = element_line(linewidth = 0.2, color = "black"),
    axis.ticks.length = unit(-3, "pt"),
    axis.title.y = element_markdown(size = 8),
    axis.title.x = element_text(size = 8),
    axis.text.x = element_text(margin = margin(t = 3), size = 7),
    axis.text.y = element_text(margin = margin(r = 3), size = 7),
    strip.text.x = element_markdown(margin = margin()),
    strip.text.y = element_markdown(margin = margin(), size = 8),
    strip.background = element_blank()
  )

ggsave("n2-scatterplots.png", width = 5.25, height = 6.45)
```