---
layout: post
title: "How to create a radial volcano plot in R and why you shouldn\'t! (CC414)"
blurb: "Volcano plots, multiple ways"
author: "PD Schloss"
date: 2026-04-08 12:00
comments: false
youtube: wQDQ_Se90WQ
start_hash: 
end_hash: 
---

In this livestream, Pat recreates and then refactors a radial volcano plot from a paper published in the journal Nature Microbiology. He created the figure using R, readxl, dplyr, ggplot2, ggtext, showtext, and other tools from the tidyverse. The functions he used from these packages included aes, annotate, arrange, arrow, bind_rows, case_when, coord_radial, download.file, element_line, element_markdown, element_text, facet_wrap, factor, filter, font_add_google, geom_hline, geom_label, geom_point, geom_richtext, ggplot, ggproto, ggsave, labeller, labs, library, list, log10, margin, mutate, read_excel, scale_alpha_manual, scale_fill_manual, scale_size_manual, scale_x_continuous, scale_y_continuous, select, seq, showtext_auto, showtext_opts, theme, tibble, and unit. The original Nature Microbiology article can be [found here](https://www.nature.com/articles/s41564-026-02289-4). His critique of the visualization can be [seen here](https://youtu.be/jmymhknbLT0). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Remake of original

```R
library(tidyverse)
library(readxl)
library(ggtext)
library(showtext)

font_add_google("Nunito", "nunito")
showtext_opts(dpi = 300)
showtext_auto()

pretty_label <- c(
  e = "<sup>3myc</sup>TurboID-<span style = 'color:#941650'>VEX2</span><br>log<sub>10</sub>(*P*)",
  f = "<span style = 'color:#941650'>VEX2</span>-TurboID<sup>3myc</sup><br>log<sub>10</sub>(*P*)",
  g = "<span style = 'color:#029293'>VEX1</span>-TurboID<sup>3myc</sup><br>log<sub>10</sub>(*P*)"
)

caf1a <- "Tb427_080044600.1-p1"
caf1b <- "Tb427_100076500.1-p1"
caf1c <- "Tb427_110054800.1-p1"
vex1 <- "Tb427_110189100.1-p1"
vex2 <- "Tb427_110151600.1-p1"

validated <- c("Tb427_100080200.1-p1", "Tb427_040015700.1-p1",
               "Tb427_010006100.1-p1", "Tb427_090092900.1-p1",
               "Tb427_000749000.1-p1", "Tb427_010014400.1-p1",
               "Tb427_100123600.1-p1", "Tb427_100127700.1-p1",
               "Tb427_040036400.1-p1", "Tb427_050017900.1-p1",
               "Tb427_050017900.1-p1", "Tb427_100071500.1-p1",
               "Tb427_070036200.1-p1", "Tb427_090044300.1-p1",
               "Tb427_110106000.1-p1",
               "Tb427_100167000.1-p1;Tb427_020021300.1-p1")


download.file("https://static-content.springer.com/esm/art%3A10.1038%2Fs41564-026-02289-4/MediaObjects/41564_2026_2289_MOESM4_ESM.xlsx",
              "figure1.xlsx")

bind_rows(e = read_excel("figure1.xlsx", sheet = "1e"),
          f = read_excel("figure1.xlsx", sheet = "1f"),
          g = read_excel("figure1.xlsx", sheet = "1g"), 
          .id = "panel") %>%
  select(panel, gene_id = GeneID, logFC,
         pvalue = P.Value, adj.pvalue = adj.P.Val) %>%
  filter(logFC >= 0) %>%
  mutate(protein_group = case_when(gene_id %in% validated ~ "validated",
                                   gene_id %in% c(caf1a, caf1b, caf1c) ~ "caf",
                                   gene_id == vex1 ~ "vex1",
                                   gene_id == vex2 ~ "vex2",
                                   pvalue < 0.0025 ~ "significant",
                                   TRUE ~ "non_significant"),
         protein_group = factor(protein_group,
                                levels = c("non_significant", "vex1", "vex2",
                                           "caf", "significant", "validated"))
         ) %>%
  arrange(protein_group) %>%
  ggplot(aes(x = logFC, y = log10(pvalue),
             fill = protein_group, size = protein_group, alpha = protein_group)
         ) +
  geom_hline(yintercept = log10(0.0025), linewidth = 0.4, linetype = "dashed") +
  geom_point(shape = 21, color = "black", stroke = 0.1, show.legend = TRUE) +
  annotate(geom = "segment",
           x = 2.33, xend = 2.67, y = 2.25, linewidth = 0.5, color = "gray40",
           arrow = arrow(angle = 30, length = unit(3, "pt"), type = "closed")) +
  facet_wrap(~panel, nrow = 1,
             labeller = labeller(panel = pretty_label)) +
  ggproto(NULL,
          coord_radial(expand = FALSE, clip = "off",
                       thetalim = c(0, 7.15), rlim = c(-12, 1.5),
                       r.axis.inside = TRUE),
          inner_radius = c(0, 0.45)) +
  scale_x_continuous(breaks = seq(0, 7, 1)) +
  scale_y_continuous(breaks = seq(-12, 1, 2),
                     labels = c("", seq(-10, -2, 2), "")) +
  scale_fill_manual(name = "VEX known interactors",
                    values = c("validated" = "#FF7E7A", "caf" = "#FEC102",
                               "vex1" = "#029293", "vex2" = "#941650",
                               "significant" = "#B3A2C8",
                               "non_significant" = "#868686"),
                    breaks = c("vex1", "vex2", "caf"),
                    labels = c("", "", ""),
                    guide = guide_legend(override.aes = list(size = 4),
                                         order = 1)) +
  scale_size_manual(name = "Enriched in +biotin samples\n(P < 0.05, FDR 1%)",
                    values = c("validated" = 3, "caf" = 3, "vex1" = 3, 
                               "vex2" = 3, "significant" = 2,
                               "non_significant" = 1),
                    breaks = "significant", 
                    labels = "",
                    guide = guide_legend(override.aes = list(size = 4,
                                                            fill = "#B3A2C8"),
                                         order = 2)
                    
                    ) +
  scale_alpha_manual(name = "ESB, SLAB, NUFIP body\nComponents validated in this study",
                    values = c("validated" = 1, "caf" = 1, "vex1" = 1, 
                               "vex2" = 1, "significant" = 1,
                               "non_significant" = 1),
                    breaks = "validated",
                    labels = "",
                    guide = guide_legend(override.aes = list(size = 4,
                                                             fill = "#FF7E7A"),
                                         order = 3
                                         )
                    ) +
  labs(x = NULL, y = NULL) +
  geom_richtext(
    data = tibble(x = 2.5, y = 2.5, label = "log<sub>2</sub>(FC)<br>(+B/WT)"),
    mapping = aes(x = x,  y = y, label = label), color = "gray40",
    inherit.aes = FALSE, hjust = 0, vjust = 1,
    label.color = NA, fill = NA, size = 2.6, label.padding = unit(0, "pt"),
    family = "nunito"
  ) +
  geom_hline(yintercept = 1.5, linewidth = 0.2, color = "gray") +
  geom_label(
    data = tibble(x = c(0, 2, 4, 6), y = 1.5, label = x),
    aes(x = x, y = y, label = label), color = "black", 
    family = "nunito", size = 7, size.unit = "pt", border.color = NA,
    label.padding = unit(1, "pt"),
    inherit.aes = FALSE
  ) +
  theme(
    text = element_text(family = "nunito"),
    plot.margin = margin(t = 3, b = 3),
    
    panel.grid.major = element_line(linewidth = 0.2, color = "gray"),
    panel.grid.minor.y = element_blank(),
    panel.background = element_blank(),
    panel.spacing.x = unit(0, "pt"),
    
    axis.ticks = element_blank(),
    axis.text.r = element_text(size = 6.5, margin = margin(r = -3), hjust = 0),
    axis.text.theta = element_blank(),
    
    strip.text = element_markdown(size = 8, margin = margin(0, 0, -5, 0)),
    strip.background = element_blank(),
    strip.clip = "off",
    
    legend.position = "bottom",
    legend.title.position = "right",
    legend.box.spacing = unit(0, "pt"),
    legend.title = element_text(size = 8, margin = margin(l = 1)),
    legend.key.spacing = unit(-4, "pt"),
    legend.text = element_blank(),
    legend.margin = margin(),
    legend.background = element_blank()
  )

ggsave("radial-volcano.png", width = 7.5, height = 2.73)
```

## Refactor to invert radius axis

```R
library(tidyverse)
library(readxl)
library(ggtext)
library(showtext)

font_add_google("Nunito", "nunito")
showtext_opts(dpi = 300)
showtext_auto()

pretty_label <- c(
  e = "<sup>3myc</sup>TurboID-<span style = 'color:#941650'>VEX2</span><br>-log<sub>10</sub>(*P*<sub>adjusted</sub>)",
  f = "<span style = 'color:#941650'>VEX2</span>-TurboID<sup>3myc</sup><br>-log<sub>10</sub>(*P*<sub>adjusted</sub>)",
  g = "<span style = 'color:#029293'>VEX1</span>-TurboID<sup>3myc</sup><br>-log<sub>10</sub>(*P*<sub>adjusted</sub>)"
)

caf1a <- "Tb427_080044600.1-p1"
caf1b <- "Tb427_100076500.1-p1"
caf1c <- "Tb427_110054800.1-p1"
vex1 <- "Tb427_110189100.1-p1"
vex2 <- "Tb427_110151600.1-p1"

validated <- c("Tb427_100080200.1-p1", "Tb427_040015700.1-p1",
               "Tb427_010006100.1-p1", "Tb427_090092900.1-p1",
               "Tb427_000749000.1-p1", "Tb427_010014400.1-p1",
               "Tb427_100123600.1-p1", "Tb427_100127700.1-p1",
               "Tb427_040036400.1-p1", "Tb427_050017900.1-p1",
               "Tb427_050017900.1-p1", "Tb427_100071500.1-p1",
               "Tb427_070036200.1-p1", "Tb427_090044300.1-p1",
               "Tb427_110106000.1-p1",
               "Tb427_100167000.1-p1;Tb427_020021300.1-p1")


download.file("https://static-content.springer.com/esm/art%3A10.1038%2Fs41564-026-02289-4/MediaObjects/41564_2026_2289_MOESM4_ESM.xlsx",
              "figure1.xlsx")

bind_rows(e = read_excel("figure1.xlsx", sheet = "1e"),
          f = read_excel("figure1.xlsx", sheet = "1f"),
          g = read_excel("figure1.xlsx", sheet = "1g"), 
          .id = "panel") %>%
  select(panel, gene_id = GeneID, logFC,
         pvalue = adj.P.Val) %>%
  filter(logFC >= 0) %>%
  mutate(protein_group = case_when(gene_id %in% validated ~ "validated",
                                   gene_id %in% c(caf1a, caf1b, caf1c) ~ "caf",
                                   gene_id == vex1 ~ "vex1",
                                   gene_id == vex2 ~ "vex2",
                                   pvalue < 0.05 ~ "significant",
                                   TRUE ~ "non_significant"),
         protein_group = factor(protein_group,
                                levels = c("non_significant", "vex1", "vex2",
                                           "caf", "significant", "validated"))
  ) %>%
  arrange(protein_group) %>%
  ggplot(aes(x = logFC, y = -log10(pvalue),
             fill = protein_group, size = protein_group, alpha = protein_group)
  ) +
  geom_hline(yintercept = -log10(0.05), linewidth = 0.4, linetype = "dashed") +
  geom_point(shape = 21, color = "black", stroke = 0.1, show.legend = TRUE) +
  annotate(geom = "segment",
           x = 2.33, xend = 2.67, y = 9.5, linewidth = 0.5, color = "gray40",
           arrow = arrow(angle = 30, length = unit(3, "pt"), type = "closed")) +
  facet_wrap(~panel, nrow = 1,
             labeller = labeller(panel = pretty_label)) +
  ggproto(NULL,
          coord_radial(expand = FALSE, clip = "off",
                       thetalim = c(0, 7.15), rlim = c(0, 9),
                       r.axis.inside = TRUE),
          inner_radius = c(0, 0.45)) +
  scale_x_continuous(breaks = seq(0, 7, 1)) +
  scale_y_continuous(breaks = seq(2, 12, 2),
                     labels = seq(2, 12, 2)) +
  scale_fill_manual(name = "VEX known interactors",
                    values = c("validated" = "#FF7E7A", "caf" = "#FEC102",
                               "vex1" = "#029293", "vex2" = "#941650",
                               "significant" = "#B3A2C8",
                               "non_significant" = "#868686"),
                    breaks = c("vex1", "vex2", "caf"),
                    labels = c("", "", ""),
                    guide = guide_legend(override.aes = list(size = 4),
                                         order = 1)) +
  scale_size_manual(name = "Enriched in +biotin samples\n(P < 0.05, FDR 1%)",
                    values = c("validated" = 3, "caf" = 3, "vex1" = 3, 
                               "vex2" = 3, "significant" = 2,
                               "non_significant" = 1),
                    breaks = "significant", 
                    labels = "",
                    guide = guide_legend(override.aes = list(size = 4,
                                                             fill = "#B3A2C8"),
                                         order = 2)
                    
  ) +
  scale_alpha_manual(name = "ESB, SLAB, NUFIP body\nComponents validated in this study",
                     values = c("validated" = 1, "caf" = 1, "vex1" = 1, 
                                "vex2" = 1, "significant" = 1,
                                "non_significant" = 1),
                     breaks = "validated",
                     labels = "",
                     guide = guide_legend(override.aes = list(size = 4,
                                                              fill = "#FF7E7A"),
                                          order = 3
                     )
  ) +
  labs(x = NULL, y = NULL) +
  geom_richtext(
    data = tibble(x = 2.5, y = 10, label = "log<sub>2</sub>(FC)<br>(+B/WT)"),
    mapping = aes(x = x,  y = y, label = label), color = "gray40",
    inherit.aes = FALSE, hjust = 0, vjust = 1,
    label.color = NA, fill = NA, size = 2.6, label.padding = unit(0, "pt"),
    family = "nunito"
  ) +
  geom_hline(yintercept = 9, linewidth = 0.2, color = "gray") +
  geom_label(
    data = tibble(x = c(0, 2, 4, 6), y = 9, label = x),
    aes(x = x, y = y, label = label), color = "black", 
    family = "nunito", size = 7, size.unit = "pt", border.color = NA,
    label.padding = unit(1, "pt"),
    inherit.aes = FALSE
  ) +
  theme(
    text = element_text(family = "nunito"),
    plot.margin = margin(t = 3, b = 3),
    
    panel.grid.major = element_line(linewidth = 0.2, color = "gray"),
    panel.grid.minor.y = element_blank(),
    panel.background = element_blank(),
    panel.spacing.x = unit(0, "pt"),
    
    axis.ticks = element_blank(),
    axis.text.r = element_text(size = 6.5, margin = margin(r = -2)),
    axis.text.theta = element_blank(),
    
    strip.text = element_markdown(size = 8, margin = margin(0, 0, -5, 0)),
    strip.background = element_blank(),
    strip.clip = "off",
    
    legend.position = "bottom",
    legend.title.position = "right",
    legend.box.spacing = unit(0, "pt"),
    legend.title = element_text(size = 8, margin = margin(l = 1)),
    legend.key.spacing = unit(-4, "pt"),
    legend.text = element_blank(),
    legend.margin = margin(),
    legend.background = element_blank()
  )

ggsave("radial-volcano_invert.png", width = 7.5, height = 2.73)
```


## Cartesian volcano plot

```R
library(tidyverse)
library(readxl)
library(ggtext)
library(showtext)

font_add_google("Nunito", "nunito")
showtext_opts(dpi = 300)
showtext_auto()

pretty_label <- c(
  e = "<sup>3myc</sup>TurboID-<span style = 'color:#941650'>VEX2</span>",
  f = "<span style = 'color:#941650'>VEX2</span>-TurboID<sup>3myc</sup>",
  g = "<span style = 'color:#029293'>VEX1</span>-TurboID<sup>3myc</sup>"
)

caf1a <- "Tb427_080044600.1-p1"
caf1b <- "Tb427_100076500.1-p1"
caf1c <- "Tb427_110054800.1-p1"
vex1 <- "Tb427_110189100.1-p1"
vex2 <- "Tb427_110151600.1-p1"

validated <- c("Tb427_100080200.1-p1", "Tb427_040015700.1-p1",
               "Tb427_010006100.1-p1", "Tb427_090092900.1-p1",
               "Tb427_000749000.1-p1", "Tb427_010014400.1-p1",
               "Tb427_100123600.1-p1", "Tb427_100127700.1-p1",
               "Tb427_040036400.1-p1", "Tb427_050017900.1-p1",
               "Tb427_050017900.1-p1", "Tb427_100071500.1-p1",
               "Tb427_070036200.1-p1", "Tb427_090044300.1-p1",
               "Tb427_110106000.1-p1",
               "Tb427_100167000.1-p1;Tb427_020021300.1-p1")


download.file("https://static-content.springer.com/esm/art%3A10.1038%2Fs41564-026-02289-4/MediaObjects/41564_2026_2289_MOESM4_ESM.xlsx",
              "figure1.xlsx")

bind_rows(e = read_excel("figure1.xlsx", sheet = "1e"),
          f = read_excel("figure1.xlsx", sheet = "1f"),
          g = read_excel("figure1.xlsx", sheet = "1g"), 
          .id = "panel") %>%
  select(panel, gene_id = GeneID, logFC,
         pvalue = adj.P.Val) %>%
  filter(logFC >= 0) %>%
  mutate(protein_group = case_when(gene_id %in% validated ~ "validated",
                                   gene_id %in% c(caf1a, caf1b, caf1c) ~ "caf",
                                   gene_id == vex1 ~ "vex1",
                                   gene_id == vex2 ~ "vex2",
                                   pvalue < 0.05 ~ "significant",
                                   TRUE ~ "non_significant"),
         protein_group = factor(protein_group,
                                levels = c("non_significant", "vex1", "vex2",
                                           "caf", "significant", "validated"))
  ) %>%
  arrange(protein_group) %>%
  ggplot(aes(x = logFC, y = -log10(pvalue),
             fill = protein_group, size = protein_group, alpha = protein_group)
  ) +
  geom_hline(yintercept = 0, color = "black", linewidth = 0.3) +
  geom_vline(xintercept = 0, color = "black", linewidth = 0.3) + 
  geom_hline(yintercept = -log10(0.05), linewidth = 0.3, linetype = "dashed",
             color = "gray40") +
  geom_point(shape = 21, color = "black", stroke = 0.1, show.legend = TRUE) +
  facet_wrap(~panel, nrow = 1,
             labeller = labeller(panel = pretty_label)) +
  coord_cartesian(expand = FALSE, clip = "off") +
  scale_x_continuous(breaks = seq(0, 7, 1), limits = c(0, 7.25)) +
  scale_y_continuous(breaks = seq(0, 9, 2),
                     labels = seq(0, 9, 2), limits = c(0, 9)) +
  scale_fill_manual(name = "VEX known interactors",
                    values = c("validated" = "#FF7E7A", "caf" = "#FEC102",
                               "vex1" = "#029293", "vex2" = "#941650",
                               "significant" = "#B3A2C8",
                               "non_significant" = "#868686"),
                    breaks = c("vex1", "vex2", "caf"),
                    labels = c("", "", ""),
                    guide = guide_legend(override.aes = list(size = 4),
                                         order = 1)) +
  scale_size_manual(name = "Enriched in +biotin samples<br>(*P*<sub>adjusted</sub> < 0.05)",
                    values = c("validated" = 3, "caf" = 3, "vex1" = 3, 
                               "vex2" = 3, "significant" = 2,
                               "non_significant" = 1),
                    breaks = "significant", 
                    labels = "",
                    guide = guide_legend(override.aes = list(size = 4,
                                                             fill = "#B3A2C8"),
                                         order = 2)
                    
  ) +
  scale_alpha_manual(name = "ESB, SLAB, NUFIP body<br>Components validated in this study",
                     values = c("validated" = 1, "caf" = 1, "vex1" = 1, 
                                "vex2" = 1, "significant" = 1,
                                "non_significant" = 1),
                     breaks = "validated",
                     labels = "",
                     guide = guide_legend(override.aes = list(size = 4,
                                                              fill = "#FF7E7A"),
                                          order = 3
                     )
  ) +
  labs(x = "log<sub>2</sub>(FC) (+B/WT)", 
       y = "-log<sub>10</sub>(*P*<sub>adjusted</sub>)") +
  theme(
    text = element_text(family = "nunito"),
    plot.margin = margin(t = 3, b = 3, l = 3, r = 3),
    
    panel.grid.major = element_line(linewidth = 0.2, color = "gray"),
    panel.grid.minor.y = element_blank(),
    panel.background = element_blank(),
    panel.spacing.x = unit(10, "pt"),
    
    axis.ticks = element_blank(),
    axis.text.x = element_text(size = 6.5, color = "black"),
    axis.text.y = element_text(size = 6.5, color = "black"),
    axis.title.x = element_markdown(size = 8, color = "black"),
    axis.title.y = element_markdown(size = 8, color = "black"),
    # axis.line = element_line(color = "black", linewidth = 0.3),
    
    strip.text = element_markdown(size = 8),
    strip.background = element_blank(),
    strip.clip = "off",
    
    legend.position = "bottom",
    legend.title.position = "right",
    legend.box.spacing = unit(5, "pt"),
    legend.title = element_markdown(size = 8, margin = margin(l = 1)),
    legend.key.spacing = unit(-4, "pt"),
    legend.text = element_blank(),
    legend.margin = margin(),
    legend.background = element_blank()
  )

ggsave("cartesian-volcano.png", width = 7.5, height = 2.73)
```
