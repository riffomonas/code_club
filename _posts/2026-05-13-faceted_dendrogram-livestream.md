---
layout: post
title: "Recreating a set of dendrorgrams, heatmaps, and a stacked bar plot from Nature Cancer (CC423)"
blurb: "Fun with patchwork, ggdendro, and ggplot2"
author: "PD Schloss"
date: 2026-05-13 12:00
comments: false
youtube: tKjpwINieRg
start_hash: 
end_hash: 
---

In this livestream, Pat recreates a set of dendrograms, heatmaps, and stacked bar plots from a paper recently published in Nature Cancer. He created the figure using R, readxl, dplyr, ggplot2, ggtext, patchwork, ggdendro, and other tools from the tidyverse. The functions he used from these packages included. The original Nature Microbiology article can be [found here](https://www.nature.com/articles/s43018-026-01136-z). His critique of the visualization can be seen at https://www.youtube.com/watch?v=rEd47kD0BWk. If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```
library(tidyverse)
library(readxl)
library(ggtext)
library(ggdendro)
library(patchwork)

facet_x <- function(x) {
  case_when(x == 6 ~ x + 0.5,
            x > 6 & x <= 10 ~ x + 1,
            x > 10 & x <= 17 ~ x + 1.5,
            x > 17 & x <= 21 ~ x + 2,
            TRUE ~ x)
}

download.file("https://static-content.springer.com/esm/art%3A10.1038%2Fs43018-026-01136-z/MediaObjects/43018_2026_1136_MOESM3_ESM.xlsx",
              "figure_1.xlsx")

panel_e <- read_excel("figure_1.xlsx", sheet = "(E)") %>%
  rename(subpopulation = "...1",
         "B<sub>prol</sub> nhood" = "B prol niche") %>%
  mutate(
    subpopulation = replace_values(
      subpopulation,
      "PD1+ TTOX" ~ "PD-1<sup>+</sup> T<sub>tox</sub>",
      "TTOX memory" ~ "T<sub>tox</sub> memory",
      "CD8T_naive" ~ "Naive CD8<sup>+</sup> T",
      "TPR" ~ "T<sub>prol</sub>",
      "TREG" ~ "T<sub>reg</sub>",
      "TH_memory" ~ "T<sub>H</sub> memory",
      "CD4T_naive" ~ "Naive CD4<sup>+</sup> T",
      "TFH" ~ "T<sub>FH</sub>",
      "B_prol" ~ "B<sub>prol</sub>"
      ))

left <- panel_e %>%
  column_to_rownames("subpopulation") %>%
  t() %>%
  dist() %>%
  hclust() %>%
  as.dendrogram() %>%
  reorder(wts = c(1, 6, 2, 4, 3, 5), agglo.FUN = mean) %>%
  dendro_data()

left_labels <- label(left)

left_dendro <- segment(left) %>%
  ggplot(aes(x = y, y = x, xend = yend, yend = xend)) +
  geom_segment(linewidth = 0.3) +
  scale_x_continuous(transform = "reverse", expand = expansion()) +
  scale_y_continuous(breaks = left_labels$x, labels = left_labels$label,
                     expand = expansion(add = 0.5), position = "right") +
  coord_cartesian(clip = "off") +
  labs(x = NULL, y = NULL)


top <- panel_e %>%
  column_to_rownames("subpopulation") %>%
  dist(method = "euclidean") %>%
  hclust(method = "complete") %>%
  as.dendrogram() %>%
  reorder(wts = c(18, 20, 5, 17, 12,
                  9, 21, 1, 2, 7,
                  4, 8, 3, 16, 6,
                  19, 15, 13, 14, 10, 11), agglo.FUN = mean) %>%
  dendro_data()

top_labels <- label(top)

top_dendro <- segment(top) %>%
  mutate(x = facet_x(x),
         xend = facet_x(xend)) %>%
  ggplot(aes(x = x, y = y, xend = xend, yend = yend)) +
  geom_segment(linewidth = 0.3) +
  scale_y_continuous(expand = expansion()) +
  scale_x_continuous(breaks = facet_x(top_labels$x), labels = top_labels$label,
                     expand = expansion(add = 0.5)) +
  coord_cartesian(clip = "off") +
  labs(x = NULL, y = NULL)



hm <- panel_e %>%
  pivot_longer(-subpopulation, values_to = "or", names_to = "neighborhood") %>%
  mutate(or  = case_when(or < -4.2 ~ -4.2,
                         or > 4.2 ~ 4.2,
                         TRUE ~ or),
         neighborhood = factor(neighborhood, levels = left_labels$label),
         subpopulation = factor(subpopulation, levels = top_labels$label)) %>%
  ggplot(aes(x = facet_x(as.numeric(subpopulation)), y = neighborhood, 
             fill = or)) +
  geom_tile(color = "black") +
  labs(x = NULL, y = NULL) +
  coord_cartesian(clip = "off", expand = FALSE) +
  scale_y_discrete(position = "right") +
  scale_x_continuous(breaks = facet_x(top_labels$x), 
                     labels = top_labels$label) +
  scale_fill_gradient2(
    name = "Enrichment<br>(log<sub>2</sub>(OR))",
    low = "#027EA8", high = "#AF2D24", mid = "white",
    limits = c(-4.2, 4.2), breaks = c(-4, 0, 4),
    guide = guide_colorbar(frame.colour = "black", frame.linewidth = 0.2)) +
  theme(
    legend.direction = "horizontal",
    legend.position = "inside",
    legend.position.inside = c(0.05, -0.45),
    legend.background = element_blank(),
    legend.title = element_markdown(size = 7, hjust = 0.5,
                                    margin = margin(r = 2)),
    legend.title.position = "left",
    legend.key.size = unit(8, "pt"),
    legend.text = element_text(size = 7, margin = margin(t = 2)),
    legend.key.spacing = unit(0, "pt"),
    legend.ticks = element_line(color = "black", linewidth = 0.2)
  )


f_left_lookup <- tribble(
  ~"region", ~"entity", ~"patient", ~"sample",
  "191_4reg006", "DLBCL", "DLBCL-10", "DLBCL 191_4reg006",
  "191_4reg007", "DLBCL", "DLBCL-10", "DLBCL 191_4reg007",
  "191_4reg004", "DLBCL", "DLBCL-11", "DLBCL 191_4reg004",
  "191_4reg005", "DLBCL", "DLBCL-11", "DLBCL 191_4reg005",
  "191_4reg002", "DLBCL", "DLBCL-2", "DLBCL 191_4reg002",
  "191_4reg003", "DLBCL", "DLBCL-2", "DLBCL 191_4reg003",
  "191_3reg001", "DLBCL", "DLBCL-9", "DLBCL 191_3reg001",
  "191_3reg004", "DLBCL", "DLBCL-9", "DLBCL 191_3reg004",
  "191_5reg005", "FL", "FL-4", "FL 191_5reg005",
  "191_5reg006", "FL", "FL-4", "FL 191_5reg006",
  "191_5reg003", "FL", "FL-5", "FL 191_5reg003",
  "191_5reg004", "FL", "FL-5", "FL 191_5reg004",
  "191_2reg001", "FL", "FL-8", "FL 191_2reg001",
  "191_2reg004", "FL", "FL-8", "FL 191_2reg004",
  "191_1reg001", "FL", "FL-7", "FL 191_1reg001",
  "191_1reg002", "FL", "FL-7", "FL 191_1reg002",
  "191_3reg002", "FL", "FL-9", "FL 191_3reg002",
  "191_3reg003", "FL", "FL-9", "FL 191_3reg003",
  "191_4reg001", "rLN", "rLN-3", "rLN 191_4reg001",
  "191_5reg002", "rLN", "rLN-5", "rLN 191_5reg002",
  "191_5reg001", "rLN", "rLN-5", "rLN 191_5reg001",
  "191_1reg006", "rLN", "rLN-6", "rLN 191_1reg006",
  "191_3reg007", "rLN", "rLN-7", "rLN 191_3reg007",
  "191_3reg008", "rLN", "rLN-7", "rLN 191_3reg008"
)

f_left_data <- read_excel("figure_1.xlsx", sheet = "(F)", range = "A2:C146") %>%
  rename_all(tolower) %>%
  rename(neighborhood = nhood) %>%
  mutate(neighborhood = replace_values(
    neighborhood,
    "B prol niche" ~ "B<sub>prol</sub> nhood")) %>%
  inner_join(., f_left_lookup, by = "sample") %>%
  summarize(fraction = sum(fraction),
            .by = c(neighborhood, patient, entity)) %>%
  mutate(neighborhood = factor(neighborhood, levels = left_labels$label))

f_left <- f_left_data %>% 
  ggplot(aes(x = fraction, y = neighborhood, fill = patient, color = patient,
             linewidth = patient)) +
  geom_col() +
  labs(x = "Proportion of cells from neighborhood", y = NULL) +
  coord_cartesian(expand = FALSE, clip = "off") +
  scale_x_continuous(limits = c(0, 1)) +
  scale_fill_manual(
    name = "rLN",
    values = c("rLN-3" = "#B597C6", "rLN-5" = "#B279B6", "rLN-6" = "#9164A1",
               "rLN-7" = "#71257C", "FL-4" = "#B9D9EC", "FL-5" = "#86B0D4",
               "FL-8" = "#2F8DBD", "FL-7" = "#2F8DBD", "FL-9" = "#054F7A", 
               "DLBCL-10" = "#9CCC65", "DLBCL-11" = "#7DB444", 
               "DLBCL-2" = "#679F37", "DLBCL-9" = "#55882F"),
    breaks = c("rLN-3", "rLN-5", "rLN-6", "rLN-7" ),
    guide = guide_legend(
      order = 1,
      override.aes = list(color = "black",
                          linewidth = 0.3)
    )
  ) +
  scale_color_manual(
    name = "FL",
    values = c("rLN-3" = "black", "rLN-5" = "black", "rLN-6" = "black",
               "rLN-7" = "black", "FL-4" = "black", "FL-5" = "black",
               "FL-8" = "black", "FL-7" = "black", "FL-9" = "black", 
               "DLBCL-10" = "black", "DLBCL-11" = "black", 
               "DLBCL-2" = "black", "DLBCL-9" = "black"),
    breaks = c("FL-4", "FL-5", "FL-8", "FL-7", "FL-9"),
    guide = guide_legend(
      order = 2,
      override.aes = list(linewidth = 0.3,
                          fill = c("FL-4" = "#B9D9EC", "FL-5" = "#86B0D4",
                                   "FL-8" = "#2F8DBD", "FL-7" = "#2F8DBD",
                                   "FL-9" = "#054F7A"))
    )
  )  +
  scale_linewidth_manual(
    name = "DLBCL",
    values = c("rLN-3" = 0.3, "rLN-5" = 0.3, "rLN-6" = 0.3,
               "rLN-7" = 0.3, "FL-4" = 0.3, "FL-5" = 0.3,
               "FL-8" = 0.3, "FL-7" = 0.3, "FL-9" = 0.3, 
               "DLBCL-10" = 0.3, "DLBCL-11" = 0.3, 
               "DLBCL-2" = 0.3, "DLBCL-9" = 0.3),
    breaks = c("DLBCL-10", "DLBCL-11", "DLBCL-2", "DLBCL-9"),
    guide = guide_legend(
      order = 3,
      override.aes = list(color = "black",
                          fill = c("DLBCL-10" = "#9CCC65",
                                   "DLBCL-11" = "#7DB444", 
                                   "DLBCL-2" = "#679F37",
                                   "DLBCL-9" = "#55882F"))
    )
  )+
  
  theme(
    legend.text = element_blank(),
    legend.direction = "horizontal",
    legend.box = "horizontal",
    legend.title.position = "top",
    legend.title = element_text(hjust = 0.5, size = 7, margin = margin(b = 2)),
    legend.position = "inside",
    legend.position.inside = c(0.5, -0.4),
    legend.key.size = unit(10, "pt"),
    legend.key.spacing = unit(0, "pt"),
    legend.background = element_blank(),
    legend.spacing.x = unit(2, "pt")
  )

f_right <- f_left_data %>%
  mutate(entity = factor(entity, levels = c("rLN", "FL", "DLBCL"))) %>%
  summarize(fraction = sum(fraction), .by = c(neighborhood, entity)) %>%
  ggplot(aes(x = entity, y = neighborhood, fill = fraction)) +
  geom_tile(color = "black", linewidth = 0.3) +
  scale_fill_gradient(
    name = "% (per nhood)",
    low = "white", high = "maroon",
    limits = c(0, NA),
    breaks = seq(0, 1, 0.2),
    labels = seq(0, 100, 20),
    guide = guide_colorbar(frame.colour = "black", frame.linewidth = 0.2)
  ) +
  coord_cartesian(clip = "off", expand = FALSE) +
  labs(x = NULL, y = NULL) +
  theme(
    legend.position = "inside",
    legend.position.inside = c(0.55, -0.43),
    legend.direction = "horizontal",
    legend.background = element_blank(),
    legend.title = element_markdown(size = 7, hjust = 0.5,
                                    margin = margin(b = 2)),
    legend.title.position = "top",
    legend.key.size = unit(8, "pt"),
    legend.text = element_text(size = 7, margin = margin(t = 2)),
    legend.key.spacing = unit(0, "pt"),
    legend.ticks = element_line(color = "black", linewidth = 0.2)
  )




layout <-
  "
  #c##
  abde
  "

left_dendro +
  theme(
    plot.margin = margin(),
    axis.text.x = element_blank(),
    axis.ticks.x = element_blank()
  ) +
  hm +
    theme(
      plot.margin = margin(),
      axis.text.x = element_markdown(angle = 90, hjust = 1, vjust = 0.5,
                                     size = 7.5, color = "black"),
      axis.ticks.x = element_blank()
    ) +
  top_dendro +
    theme(
      plot.margin = margin(),
      axis.text.x = element_markdown(angle = 90, hjust = 1, vjust = 0.5,
                                     size = 7.5, color = "black"),
      axis.ticks.x = element_blank()
    ) +
  f_left +
    theme(
      plot.margin = margin(),
      axis.text.x = element_text(size = 6.5, color = "black"),
      axis.ticks.x = element_line(linewidth = 0.2)
    ) +
  f_right +
    theme(
      plot.margin = margin(l = 10),
      axis.text.x = element_text(size = 7.5, color = "black", angle = 90,
                                 hjust =1, vjust = 0.5),
      axis.ticks = element_blank(),
      
    ) +
  plot_layout(axes = "collect", design = layout,
              widths = c(0.05, 0.5, 0.35, 0.1), heights = c(0.2, 0.8)) &
  theme(
    axis.text.y.right = element_markdown(hjust = 0.5, size = 7.5,
                                         margin = margin(l = 3, r = 3), 
                                         color = "black"),
    axis.text.y = element_blank(),
    axis.ticks.y = element_blank(),
    axis.title.x = element_text(size = 8, margin = margin(t = -35)),
    panel.grid = element_blank(),
    panel.background = element_blank()
  )

ggsave("figure_1ef.png", width = 7.5, height = 2.5)
```