---
layout: post
title: "How to insert images into ggplot2 generated figures in R (CC390)"
blurb: "Let's add some jitter..."
author: "PD Schloss"
date: 2025-12-17 12:00
comments: false
youtube: cfxcnWfsgmE
start_hash: 
end_hash: 
---

Pat recreates a set of four panels that had a cartoon annotation, jittered points, and a. line through their median. The original panels were included as part of an article published in Nature Microbiology. He created the figure using R, dplyr, ggplot2, ggtext, readxl, patchwork, and other tools from the tidyverse. The functions he used from these packages included aes, bind_rows, coord_cartesian, download.file, drop_na, element_line, element_markdown, element_text, factor, filter, font_add_google, function, geom_hline, geom_jitter, geom_richtext, ggplot, ggsave, glue, labs, library, list, margin, mutate, pivot_longer, plot_annotation, plot_layout, position_jitter, read_excel, scale_color_manual, scale_fill_manual, scale_x_continuous, scale_y_log10, separate_wider_delim, showtext_auto, showtext_opts, stat_summary, theme, theme_classic, tribble, and unit. The newsletter describing this visualization at a 30,000 ft view can be [found here](https://shop.riffomonas.org/posts/do-you-want-to-up-your-data-visualization-designs-in-2026). You can find the original article presenting the figure at [*Nature Microbiology*](https://www.nature.com/articles/s41564-025-02162-w). Here's [a video critiquing](https://riffomonas.org/code_club/2025-12-08-invader-displacement-critique) the original figure. Here's a [video showing the recreation of panels f and g](https://riffomonas.org/code_club/2025-12-10-invader-displacement). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

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

panel_title <- tribble(
  ~panel, ~text,
  "b", "Metabolic overlap<br>No toxins",
  "c", "Invader private nutrient<br>No toxins",
  "d", "Metabolic overlap<br>Invader with toxin",
  "e", "Invader private nutrient<br>Invader with toxin"
) %>%
    mutate(cartoon = glue("<img src = 'cartoon_{panel}.png' height = '30'>"))

panels_b_e_data <- bind_rows(
  b = read_excel(xlsx_file, range = "A5:S8",
                       col_names = c("time",
                         glue("resident_wt-{1:9}"),
                         glue("invader_wt-{1:9}"))),
  c = read_excel(xlsx_file, range = "A14:Q17",
           col_names = c("time",
                         glue("resident_ko-{1:8}"),
                         glue("invader_wt-{1:8}"))),
  d = read_excel(xlsx_file, range = "A23:K26",
           col_names = c("time",
                         glue("resident_wt-{1:5}"),
                         glue("invader_tox-{1:5}"))),
  e = read_excel(xlsx_file, range = "A32:K35",
           col_names = c("time",
                         glue("resident_ko-{1:5}"),
                         glue("invader_tox-{1:5}"))),
  .id = "panel") %>%
  pivot_longer(-c(panel, time),
               names_to = "condition_rep",
               values_to = "density") %>%
  drop_na() %>%
  separate_wider_delim(condition_rep, delim = "-",
                       names = c("condition", "rep")) %>%
  mutate(condition = factor(condition,
                            levels = c("resident_wt", "invader_wt",
                                       "resident_ko", "invader_tox")))

plot_panels_b_e <- function(p) {  
  
  panels_b_e_data %>%
    filter(panel == p) %>%
    
    ggplot(aes(x = time, y = density, fill = condition, color = condition)) +
    geom_hline(yintercept = 200, color = "gray", linewidth = 0.2) +
    stat_summary(geom = "line", fun = median, show.legend = FALSE) +
    geom_jitter(show.legend = TRUE,
      shape = 21, stroke = 0.5,
      position = position_jitter(width = 2, height = 0, seed = 19760620)) +
    geom_richtext(data = filter(panel_title, panel == p),
                  aes(x = 38, y = 6e10, label = text), hjust = 0,
                  family = "nunito", fill = NA, label.color = NA,
                  label.padding = unit(0, "pt"), size = 2,
                  inherit.aes = FALSE) +
    geom_richtext(data = filter(panel_title, panel == p),
                  aes(x = 38, y = 6e10, label = cartoon), hjust = 1,
                  fill = NA, label.color = NA,
                  label.padding = unit(0, "pt"),
                  inherit.aes = FALSE) +
    scale_color_manual(
      name = NULL,
      breaks = c("resident_wt", "invader_wt", "resident_ko", "invader_tox"),
      values = c(resident_wt = "#6298D0", invader_wt = "#E60000",
                 resident_ko = "#6298D0", invader_tox = "#E60000"),
      labels = c(resident_wt = "Resident (WT)", invader_wt = "Invader (WT)",
                 resident_ko = "Resident (&Delta;*srlAEB*)",
                 invader_tox = "Invader (WT with colicin E2)"),
      drop = FALSE
    ) +
    scale_fill_manual(
      name = NULL,
      breaks = c("resident_wt", "invader_wt", "resident_ko", "invader_tox"),
      values = c(resident_wt = "#FFFFFF", invader_wt = "#FFFFFF",
                 resident_ko = "#DDDDDD", invader_tox = "#000000"),
      labels = c(resident_wt = "Resident (WT)", invader_wt = "Invader (WT)",
                 resident_ko = "Resident (&Delta;*srlAEB*)",
                 invader_tox = "Invader (WT with colicin E2)"),
      drop = FALSE
    ) +

    scale_y_log10(
      breaks = 10^(seq(1, 9, 2)),
      labels = glue("10<sup>{seq(1, 9, 2)}</sup>")
    ) +
    scale_x_continuous(breaks = c(8, 24, 48, 72)) +
    coord_cartesian(
      ylim = c(1e1, 1e10),
      xlim = c(0, 76),
      expand = FALSE, clip = "off"
    ) +
    labs(
      x = "Time (h)",
      y = "Density (CFU ml<sup>-1</sup>)"
    ) +
    theme_classic() +
    theme(
      text = element_text(family = "nunito"),
      plot.margin = margin(t = 25, r = 8, b = 3, l = 8),
      axis.text.y = element_markdown(size = 6),
      axis.text.x = element_markdown(size = 7),
      axis.title.y = element_markdown(size = 7, margin = margin(r = 6)),
      axis.title.x = element_markdown(size = 7),
      axis.line = element_line(linewidth = 0.2),
      axis.ticks = element_line(linewidth = 0.2),
      axis.ticks.length = unit(3, "pt")
      
    )
}

plot_panels_b_e("b") +
  plot_panels_b_e("c") +
  plot_panels_b_e("d") +
  plot_panels_b_e("e") +
  plot_layout(guides = "collect") +
  plot_annotation(
            theme = theme(plot.margin = margin(0, 0, 0, 0)),
            tag_levels = list(c("b", "c", "d", "e"))) &
  theme(plot.tag = element_text(face = "bold"),
        plot.tag.position = c(0, 1.15),
        plot.tag.location = "plot",
        legend.text = element_markdown(size = 7),
        legend.key.height = unit(9, "pt"),
        legend.key.width = unit(9, "pt"),
        legend.box.spacing = unit(0, "pt"),
        legend.margin = margin(0,0,0,5)
        )

ggsave("panels_b_e.png", width = 7, height = 4.1)
```