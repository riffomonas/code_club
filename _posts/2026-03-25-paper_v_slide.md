---
layout: post
title: "How do you modify a figure from a paper to use in a presentation? (CC410)"
blurb: "Bigger and simpler"
author: "PD Schloss"
date: 2026-03-25 12:00
comments: false
youtube: xuDtr6Ky6yw
start_hash: 
end_hash: 
---

In this livestream, Pat refactors a plot from a paper his research group published in the journal MBio with an eye towards improving it as a figure in the paper, but also for use in a presentation. He created the figure using R, dplyr, ggplot2, ggtext, glue, and other tools from the tidyverse. The functions he used from these packages included aes, annotate, element_blank, element_line, element_text, element_textbox_simple, factor, filter, geom_jitter, geom_vline, ggplot, ggsave, glue, labs, library, list, margin, mean_se, mutate, plot_annotation, plot_layout, position_jitter, read_csv, rev, scale_color_manual, scale_x_continuous, select, stat_summary, str_replace, theme, and theme_bw. The original MBio article can be [found here](https://journals.asm.org/doi/10.1128/mbio.03161-21). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```R
library(tidyverse)
library(glue)
library(patchwork)
library(ggtext)

tax_levels <- c("phylum", "class", "order", "family", "genus", "otu", "asv")
tax_labels <- c("Phylum", "Class", "Order", "Family", "Genus", "OTU", "ASV")
                # "Operational\nTaxonomic\nUnit", "Amplicon\nSequence\nVariant")

auc_data <- glue("https://raw.githubusercontent.com/SchlossLab/Armour_Resolution_mBio_2021/refs/heads/master/data/process/combined-{tax_levels}-rf.csv") %>%
  read_csv(id = "level") %>%
  mutate(level = str_replace(level, ".*combined-(.*)-rf.csv", "\\1")) %>%
  select(level, seed, auc = AUC) %>%
  mutate(group_d = level %in% c("family", "genus", "otu"),
         level = factor(level, levels = rev(tax_levels),
                        labels = rev(tax_labels)),
         )

panel_a <- auc_data %>%
  ggplot(aes(x = auc, y = level, color = group_d)) +
  geom_vline(xintercept = c(0.5, 0.85), linetype = 1, linewidth = 0.5,
             color = "white") +
  geom_vline(xintercept = 0.5, linetype = "longdash",
             linewidth = 0.2, color = "gray40") +
  geom_jitter(
    color = "gray80",
    alpha = 0.5,
    position = position_jitter(height = 0.25, seed = 19760620)) +
  stat_summary(geom = "pointrange",
               fun.data = mean_se, fun.args = list(mult = 2),
               show.legend = FALSE) +
  annotate(
    geom = "text",
    x = 0.85, y = 1:7, label = c("E", "D", "D,E", "D,E", "C", "B", "A"),
    size = 7, size.unit = "pt"
  ) +
  scale_color_manual(
    values = c("FALSE" = "gray30", "TRUE" = "dodgerblue") 
  ) +
  labs(x = "Area Under the Receiver\nOperator Characteristic Curve",
       y = NULL) +
  theme_bw() +
  theme(
    plot.margin = margin(),
    panel.grid.major.y = element_blank(),
    panel.grid.major.x = element_line(linewidth = 0.2),
    panel.grid.minor.x = element_line(linewidth = 0.2),
    axis.ticks.y = element_blank(),
    axis.ticks.x = element_line(linewidth = 0.2),
    axis.title.x = element_text(size = 8),
    axis.text.y = element_text(size = 7, lineheight = 0.8, hjust = 0)
  )



sens_spec_data <- read_csv("https://raw.githubusercontent.com/SchlossLab/Armour_Resolution_mBio_2021/refs/heads/master/analysis/sens_spec_by_level.csv") %>%
  mutate(group_d = level %in% c("family", "genus", "otu"),
         level = factor(level, levels = rev(tax_levels),
                        labels = rev(tax_labels)),
         sensitivity = sensitivity * 100
  )


panel_b <- sens_spec_data %>%
  filter(specificity == 0.9) %>%
  ggplot(aes(x = sensitivity, y = level, color = group_d)) +
  geom_vline(xintercept = 65, linewidth = 0.5, color = "white") +
  geom_jitter(
    color = "gray80",
    alpha = 0.5,
    position = position_jitter(height = 0.25, seed = 19760620)) +
  stat_summary(geom = "pointrange",
               fun.data = mean_se, fun.args = list(mult = 1),
               show.legend = FALSE) +
  annotate(
    geom = "text",
    x = 65, y = 1:7, label = c("Z", "Y", "Y", "Y", "X", "W", "W"),
    size = 7, size.unit = "pt"
  ) +
  scale_color_manual(
    values = c("FALSE" = "gray30", "TRUE" = "dodgerblue") 
  ) +
  labs(x = "Sensitivity (%)\nat 90% Specificity",
       y = NULL) +
  scale_x_continuous(breaks = seq(10, 60, 10)) +
  theme_bw() +
  theme(
    plot.margin = margin(),
    panel.grid.major.y = element_blank(),
    panel.grid.major.x = element_line(linewidth = 0.2),
    panel.grid.minor.x = element_line(linewidth = 0.2),
    axis.ticks.y = element_blank(),
    axis.ticks.x = element_line(linewidth = 0.2),
    axis.title.x = element_text(size = 8),
    axis.text.y = element_text(size = 7, lineheight = 0.8, hjust = 0)
  )

ggsave("panel_b.png", width = 3, height = 3)


(panel_a + theme(plot.margin = margin(r = 10))) +
  panel_b +
  plot_layout(axes = "collect") +
  plot_annotation(tag_levels = 'A') & 
  theme(plot.tag = element_text(size = 10, face = "bold",
                                margin = margin(t = -5, l = -5)))

ggsave("figure_1.png", width = 6, height = 3)



# work on creating panel B for stand alone figure for a slide

sens_spec_data %>%
  filter(specificity == 0.9) %>%
  ggplot(aes(x = sensitivity, y = level, color = group_d)) +
  stat_summary(geom = "pointrange",
               fun.data = mean_se, fun.args = list(mult = 1),
               show.legend = FALSE, size = 1.2,  linewidth = 1) +
  scale_color_manual(
    values = c("FALSE" = "gray30", "TRUE" = "dodgerblue") 
  ) +
  labs(x = "Sensitivity (%) at 90% Specificity",
       y = NULL,
       title = "<span style='color:dodgerblue'>Family, genus, and OTU level classifications</span> had significantly better performance over other taxonomic ranks",
       caption = "Each point represents the mean of 100 random splits into a training and testing set"
       ) +
  scale_x_continuous(breaks = seq(10, 60, 5)) +
  theme_bw() +
  theme(
    plot.margin = margin(l = 60, r = 60),
    plot.title = element_textbox_simple(face = "bold", size = 20,
                                        lineheight = 0.9,
                                        margin = margin(t = 5, b = 10,
                                        l = -60, r = -60)),
    plot.title.position = "plot",
    plot.caption = element_text(hjust = 0, margin = margin(l = -60, t = 10)),
    plot.caption.position = "plot",
    panel.grid.major.y = element_blank(),
    panel.grid.major.x = element_line(linewidth = 0.2),
    panel.grid.minor.x = element_blank(),
    axis.ticks.y = element_blank(),
    axis.ticks.x = element_line(linewidth = 0.2),
    axis.title.x = element_text(size = 15),
    axis.text.y = element_text(size = 15, lineheight = 0.8, hjust = 0),
    axis.text.x = element_text(size = 12)
  )

ggsave("panel_b.png", width = 6, height = 4)
```