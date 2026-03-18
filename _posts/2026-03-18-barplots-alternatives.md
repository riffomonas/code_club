---
layout: post
title: "Creating alternatives to dynamite plots in R. Which is the best? (CC408)"
blurb: "Same figure, six appraoches"
author: "PD Schloss"
date: 2026-03-18 12:00
comments: false
youtube: JP5Z67CNUFA
start_hash: 
end_hash: 
---

In this livestream, Pat recreates a recently published dynamite plot published in Nautre and then refactors it with 6 other approaches. A dynamite plot is a bar plot with error bars showing the standard deviation and then often has data overlaid on top of it.He created the figures using R, dplyr, ggplot2, readxl, ggbeeswarm, ggh4x, and other tools from the tidyverse. The functions he used from these packages included aes, arrange, coord_cartesian, download.file, element_blank, element_line, element_text, excel_sheets, expansion, factor, geom_beeswarm, geom_boxplot, geom_point, geom_text, geom_violin, ggplot, ggsave, guide_legend, guides, labs, library, list, map, margin, mean, mean_sdl, mutate, nest, position_dodge, position_jitterdodge, read_excel, rename_all, scale_color_manual, scale_fill_manual, scale_y_continuous, stat_summary, str_remove, t.test, theme, theme_classic, tibble, tidy, unit, and unnest. The original Nature article can be [found here](https://www.nature.com/articles/s41586-026-10256-6). A video critiquing the original version of the figure can be [found here](https://youtu.be/PiUliS5sdyk). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```R
library(tidyverse)
library(readxl)
library(broom)

url <- "https://static-content.springer.com/esm/art%3A10.1038%2Fs41586-026-10256-6/MediaObjects/41586_2026_10256_MOESM5_ESM.xlsx"

download.file(url, "dynamite_data.xlsx")

excel_sheets("dynamite_data.xlsx")

panel_i_data <- read_excel("dynamite_data.xlsx", sheet = "Fig.1i") %>%
  rename_all(tolower) %>%
  mutate(
    celltype = factor(celltype, levels = c("EMPP", "BEMP", "preMegE"),
                      labels = c("EMPP", "BEMP", "PreMegE")),
    subgroup = factor(subgroup,
                      levels = c("PBS_Day2", "IL33_Day2", 
                                 "PBS_Day6", "IL33_Day6"),
                      labels = c("PBS day 2", "IL-33 day 2", 
                                 "PBS day 6", "IL-33 day 6")),
    condition = str_remove(subgroup, " day .") %>%
      factor(., levels = c("PBS", "IL-33")),
    day = str_remove(subgroup, ".*day ")
  ) %>%
  arrange(celltype, subgroup)

p_values <- panel_i_data %>%
  nest(data = -c(celltype, day)) %>%
  mutate(t = map(data, ~t.test(data~condition, data = .x) %>% tidy())) %>%
  unnest(t)


# dynamite recreation

panel_i_data %>%
  ggplot(aes(x = celltype, y = data, fill = subgroup)) +
  stat_summary(geom = "bar",
               fun = mean, color = "black", linewidth = 0.1,
               position = position_dodge()) +
  stat_summary(geom = "errorbar",
               fun.data = mean_sdl, fun.args = list(mult = 1),
               linewidth = 0.2, width = 0.4,
               position = position_dodge(width = 0.9)) +
  geom_point(position = position_dodge(width = 0.9), 
             size = 0.3, show.legend = FALSE) +
  scale_y_continuous(expand = expansion(mult = c(0, 0.05))) +
  scale_fill_manual(
    breaks = c("PBS day 2", "IL-33 day 2", 
               "PBS day 6", "IL-33 day 6"),
    values = c("#92C4DC", "#9BC287", "#0075B2", "#2C9A49")
  ) +
  labs(x = NULL, y = "Frequency of cell types\nin LK cell fraction (%)",
       fill = NULL) +
  theme_classic(
    base_size = 7,
    base_line_size = 0.2
  ) +
  theme(
    plot.margin = margin(2, 2, 2, 2),
    legend.position = "inside",
    legend.position.inside = c(0.33, 0.80),
    legend.key.size = unit(7, "pt"),
    legend.margin = margin(0, 0, 0, 0),
    legend.text = element_text(margin = margin(0, 0, 0, 2)),
    axis.ticks.x = element_blank(),
    axis.title.y = element_text(size = 7)
  )

ggsave("dynamite_recreation.png", width = 1.875, height = 1.875)


# dynamite w/ beeswarm

library(ggbeeswarm)

panel_i_data %>%
  ggplot(aes(x = celltype, y = data, fill = subgroup)) +
  stat_summary(geom = "bar",
               fun = mean, color = "black", linewidth = 0.1,
               position = position_dodge()) +
  stat_summary(geom = "errorbar",
               fun.data = mean_sdl, fun.args = list(mult = 1),
               linewidth = 0.2, width = 0.4,
               position = position_dodge(width = 0.9)) +
  geom_beeswarm(dodge.width = 0.9, cex = 1.5, size = 0.8,
                show.legend = FALSE) +
  scale_y_continuous(expand = expansion(mult = c(0, 0.05))) +
  scale_fill_manual(
    breaks = c("PBS day 2", "IL-33 day 2", 
               "PBS day 6", "IL-33 day 6"),
    values = c("#92C4DC", "#9BC287", "#0075B2", "#2C9A49")
  ) +
  labs(x = NULL, y = "Frequency of cell types\nin LK cell fraction (%)",
       fill = NULL) +
  theme_classic(
    base_size = 7,
    base_line_size = 0.2
  ) +
  theme(
    plot.margin = margin(2, 2, 2, 2),
    legend.position = "inside",
    legend.position.inside = c(0.33, 0.80),
    legend.key.size = unit(7, "pt"),
    legend.margin = margin(0, 0, 0, 0),
    legend.text = element_text(margin = margin(0, 0, 0, 2)),
    axis.ticks.x = element_blank(),
    axis.title.y = element_text(size = 7)
  )

ggsave("dynamite_beeswarm.png", width = 1.875, height = 1.875)


# dynamite w/ jitter

panel_i_data %>%
  ggplot(aes(x = celltype, y = data, fill = subgroup)) +
  stat_summary(geom = "bar",
               fun = mean, color = "black", linewidth = 0.1,
               position = position_dodge()) +
  stat_summary(geom = "errorbar",
               fun.data = mean_sdl, fun.args = list(mult = 1),
               linewidth = 0.2, width = 0.4,
               position = position_dodge(width = 0.9)) +
  geom_point(position = position_jitterdodge(dodge.width = 0.9,
                                             seed = 19760620), 
             size = 0.5, show.legend = FALSE) +
  scale_y_continuous(expand = expansion(mult = c(0, 0.05))) +
  scale_fill_manual(
    breaks = c("PBS day 2", "IL-33 day 2", 
               "PBS day 6", "IL-33 day 6"),
    values = c("#92C4DC", "#9BC287", "#0075B2", "#2C9A49")
  ) +
  labs(x = NULL, y = "Frequency of cell types\nin LK cell fraction (%)",
       fill = NULL) +
  theme_classic(
    base_size = 7,
    base_line_size = 0.2
  ) +
  theme(
    plot.margin = margin(2, 2, 2, 2),
    legend.position = "inside",
    legend.position.inside = c(0.33, 0.80),
    legend.key.size = unit(7, "pt"),
    legend.margin = margin(0, 0, 0, 0),
    legend.text = element_text(margin = margin(0, 0, 0, 2)),
    axis.ticks.x = element_blank(),
    axis.title.y = element_text(size = 7)
  )

ggsave("dynamite_jitter.png", width = 1.875, height = 1.875)


# boxplot w/ jitter

panel_i_data %>%
  ggplot(aes(x = celltype, y = data, color = subgroup)) +
  geom_boxplot(outlier.shape = NA, show.legend = FALSE) +
  geom_point(position = position_jitterdodge(dodge.width = 0.8,
                                             seed = 19760620), 
             size = 0.5) +
  scale_y_continuous(
    limits = c(0, NA),
    expand = expansion(mult = c(0, 0.05))) +
  scale_color_manual(
    breaks = c("PBS day 2", "IL-33 day 2", 
               "PBS day 6", "IL-33 day 6"),
    values = c("#92C4DC", "#9BC287", "#0075B2", "#2C9A49")
  ) +
  labs(x = NULL, y = "Frequency of cell types\nin LK cell fraction (%)",
       color = NULL) +
  theme_classic(
    base_size = 7,
    base_line_size = 0.2
  ) +
  theme(
    plot.margin = margin(2, 2, 2, 2),
    legend.position = "inside",
    legend.position.inside = c(0.33, 0.80),
    legend.key.size = unit(7, "pt"),
    legend.margin = margin(0, 0, 0, 0),
    legend.text = element_text(margin = margin(0, 0, 0, 2)),
    axis.ticks.x = element_blank(),
    axis.title.y = element_text(size = 7)
  )

ggsave("boxplot_jitter.png", width = 1.875, height = 1.875)


# violin w/ jitter

panel_i_data %>%
  ggplot(aes(x = celltype, y = data, color = subgroup)) +
  geom_violin(show.legend = FALSE) +
  # geom_point(position = position_jitterdodge(dodge.width = 0.9,
  #                                            seed = 19760620), 
  #            size = 0.5, show.legend = FALSE) +
  geom_beeswarm(dodge.width = 0.9, cex = 1.5, size = 0.8,
                show.legend = TRUE) +
  scale_y_continuous(
    limits = c(0, NA),
    expand = expansion(mult = c(0, 0.05))) +
  scale_color_manual(
    breaks = c("PBS day 2", "IL-33 day 2", 
               "PBS day 6", "IL-33 day 6"),
    values = c("#92C4DC", "#9BC287", "#0075B2", "#2C9A49")
  ) +
  labs(x = NULL, y = "Frequency of cell types\nin LK cell fraction (%)",
       color = NULL) +
  theme_classic(
    base_size = 7,
    base_line_size = 0.2
  ) +
  theme(
    plot.margin = margin(2, 2, 2, 2),
    legend.position = "inside",
    legend.position.inside = c(0.33, 0.80),
    legend.key.size = unit(7, "pt"),
    legend.margin = margin(0, 0, 0, 0),
    legend.text = element_text(margin = margin(0, 0, 0, 2)),
    axis.ticks.x = element_blank(),
    axis.title.y = element_text(size = 7)
  )

ggsave("violin_jitter.png", width = 1.875, height = 1.875)



# mean/median w/ jitter/beeswarm

panel_i_data %>%
  ggplot(aes(x = celltype, y = data, color = subgroup, group = subgroup)) +
  stat_summary(geom = "crossbar", show.legend = FALSE,
               fun = mean, linewidth = 0.2, color = "black",
               position = position_dodge()) +
  geom_point(position = position_jitterdodge(dodge.width = 0.9,
                                             seed = 19760620), 
             size = 0.5, show.legend = TRUE) +
  scale_y_continuous(
    limits = c(0, NA),
    expand = expansion(mult = c(0, 0.05))) +
  scale_color_manual(
    breaks = c("PBS day 2", "IL-33 day 2", 
               "PBS day 6", "IL-33 day 6"),
    values = c("#92C4DC", "#9BC287", "#0075B2", "#2C9A49")
  ) +
  labs(x = NULL, y = "Frequency of cell types\nin LK cell fraction (%)",
       color = NULL) +
  theme_classic(
    base_size = 7,
    base_line_size = 0.2
  ) +
  theme(
    plot.margin = margin(2, 2, 2, 2),
    legend.position = "inside",
    legend.position.inside = c(0.25, 0.80),
    legend.key.size = unit(7, "pt"),
    legend.margin = margin(0, 0, 0, 0),
    legend.text = element_text(margin = margin(0, 0, 0, 2)),
    axis.ticks.x = element_blank(),
    axis.title.y = element_text(size = 7)
  ) +
  guides(
    color = guide_legend(override.aes = list(size = 1))
  )

ggsave("jitter_meanline.png", width = 1.875, height = 1.875)


# something different...

library(ggh4x)

panel_i_data %>%
  ggplot(aes(x = condition, y = data, color = condition)) +
  stat_summary(geom = "crossbar", show.legend = FALSE,
               fun = mean, linewidth = 0.2, color = "black",
               position = position_dodge()) +
  geom_point(position = position_jitterdodge(dodge.width = 0.9,
                                             seed = 19760620), 
             size = 0.5, show.legend = TRUE) +
  scale_y_continuous(
    expand = expansion(mult = c(0, 0.05))) +
  coord_cartesian(
    ylim = c(0, NA),
    xlim = c(1, 2),
    clip = "off"
  ) +
  scale_color_manual(
    values = c("PBS" = "#0075B2", "IL-33" = "#2C9A49")
  ) +
  guides(
    color = guide_legend(override.aes = list(size = 1))
  ) +
  labs(x = NULL, y = "Frequency of cell types\nin LK cell fraction (%)",
       color = NULL) +
  facet_nested(~celltype + day, switch = "x", 
               nest_line = element_line(linewidth = 0.2)) +
  geom_text(data = tibble(data = 0, celltype = factor("EMPP"),
                          day = "2", condition = "PBS"),
            aes(label = "Day", x = 0, y = -0.7), 
            color = "black", size = 6, size.unit = "pt", hjust = 1) +
  theme_classic(
    base_size = 7,
    base_line_size = 0.2
  ) +
  theme(
    plot.margin = margin(2, 2, 0, 2),
    legend.position = "inside",
    legend.position.inside = c(0.25, 0.80),
    legend.key.size = unit(7, "pt"),
    legend.margin = margin(0, 0, 0, 0),
    legend.text = element_text(margin = margin(0, 0, 0, 2)),
    axis.ticks.x = element_blank(),
    axis.title.y = element_text(size = 7),
    axis.text.x = element_blank(),
    strip.background = element_blank(),
    strip.placement = "outside",
    strip.text = element_text(margin = margin(t = 1, b = 2))
  )

ggsave("jitter_faceted.png", width = 1.875, height = 1.875)
```