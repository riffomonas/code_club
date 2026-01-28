---
layout: post
title: "Creating a visualization that looks like a 96 well plate in R with ggplot2 (CC395)"
blurb: "Unique approach to MIC data"
author: "PD Schloss"
date: 2026-01-21 12:00
comments: false
youtube: 9OR9lueCuyE
start_hash: 
end_hash: 
---

Pat creates two plots in this livestream. First, he recreates a horizontal of panels that were presented as facets in the original figure. Then he refactors those plots to be vertically arrayed. The original panels were included as part of an article published in Nature Microbiology. He created the figure using R, dplyr, ggplot2, ggtext, readxl, showtext, and other tools from the tidyverse. The functions he used from these packages included aes, annotate, axis.ticks = element_blank, axis.title.x = element_markdown, coord_cartesian, download.file, element_blank, element_text, facet_wrap, factor, filter, font_add_google, geom_point, geom_text, ggplot, ggsave, guides, if_else, labs, library, margin, min, mutate, read_excel, round, scale_color_manual, scale_discrete_manual, scale_fill_gradient, scale_y_discrete, showtext_auto, showtext_opts, theme, and unit. The newsletter describing this visualization at a 30,000 ft view can be [found here](https://shop.riffomonas.org/posts/mic-drop-representing-antibiotic-susceptibility-as-a-stylized-heatmap-with-r-s-ggplot2). You can find the original article presenting the figure at [*Nature Microbiology*](https://www.nature.com/articles/s41564-025-02189-z). Here's a [video critiquing](https://youtu.be/SIs5zTCEQ3Q) the original figure. If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```R
library(tidyverse)
library(readxl)
library(ggtext)
library(showtext)

font_add_google("Nunito", "nunito", regular.wt = 500)
showtext_opts(dpi = 300)
showtext_auto()

xlsx_url <- "https://static-content.springer.com/esm/art%3A10.1038%2Fs41564-025-02189-z/MediaObjects/41564_2025_2189_MOESM5_ESM.xlsx"
download.file(xlsx_url, destfile = "mic_data.xlsx")

d <- read_excel("mic_data.xlsx", sheet = "F2A",
           col_names = c("strain", "amb", "co2", "growth"),
           skip = 1) %>%
  mutate(is_mic = amb == min(amb[growth < 10]), .by = c(strain, co2),
         is_mic = if_else(amb == 0, FALSE, is_mic)) %>%
  mutate(amb = factor(round(amb, digits = 3)),
         co2 = factor(100 * co2,
                      levels = c("0.04", "1.5", "5.5"),
                      labels = c("0.04% CO<sub>2</sub>", "1.5% CO<sub>2</sub>", "5.5% CO<sub>2</sub>")),
         strain = factor(
           strain,
          levels = rev(c("R1", "rca1", "efg1", "rca1efg1", "nce103",
                         "rca1nce103", "rca1eNce103", "efg1eNCE103",
                         "ptc2", "rca1ptc2", "rca1RCA1", "efg1EFG1",
                         "nce103NCE103", "S", "B8441")),
          labels = rev(c("R1", "*rca1*&Delta;", "*efg1*&Delta;",
                         "*rca1*&Delta;*efg1*&Delta;", "*nce103*&Delta;",
                         "*rca1*&Delta;*nce103*&Delta;", 
                         "*rca1*&Delta;*eNCE103*",
                         "*efg1*&Delta;*eNCE103*", "*ptc2*&Delta;",
                         "*rca1*&Delta;*ptc2*&Delta;", "*rca1*&Delta;*RCA1*",
                         "*efg1*&Delta;*EFG1*", "*nce103*&Delta;*NCE103*",
                         "S", "AR387"
          )))
         )

d %>%
  ggplot(aes(x = amb, y = strain, fill = growth,
             color = is_mic, stroke = is_mic)) +
  geom_point(shape = 21, size = 4.75) + 
  geom_text(data = filter(d, is_mic), label = "MIC", color = "#8E8E8E",
            family = "nunito", size = 5, size.unit = "pt") +
  facet_wrap(.~co2, nrow = 1) +
  annotate(geom = "segment",
           x = 11.75, y = 0.5, yend = 15.5, linewidth = 0.2,
           layout = c(1, 2)) +
  scale_fill_gradient(low = "#FFFFFF", high = "#D08C5E") +
  scale_color_manual(
    values = c("TRUE" = "#CE3769", "FALSE" = "#6C6059")
    # breaks = c(TRUE, FALSE),
    # values = c("#CE3769", "#6C6059")
  ) +
  scale_discrete_manual(
    aesthetics = "stroke",
    values = c("TRUE" = 0.8, "FALSE" = 0.5)
  ) +
  coord_cartesian(expand = FALSE, clip = "off",
                  xlim = c(0.5, 11.5),
                  ylim = c(0.5, 15.5)) +
  guides(color = "none", stroke = "none") +
  labs(x = "Amphotericin B (&mu;g ml<sup>-1</sup>)", y = NULL,
       fill = "Relative growth (%)") +
  theme(
    text = element_text(family = "nunito"),
    panel.grid = element_blank(),
    panel.background = element_blank(),
    
    # axis.title.y = element_blank(),
    axis.title.x = element_markdown(size = 9),
    axis.text.y = element_markdown(color = "black"),
    axis.text.x = element_text(angle = 45, hjust = 1, color = "black",
                               size = 7.5),
    axis.ticks = element_blank(),

    strip.text = element_markdown(margin = margin(0, 0, 3, 0)),
    strip.background = element_blank(),
    
    legend.title = element_text(angle = -90, vjust = 0.2, size = 9),
    legend.text = element_text(size = 8),
    legend.key.width = unit(15, "pt"),
    legend.key.height = unit(14, "pt"),
    legend.position = "right",
    legend.justification.right = "top",
    legend.margin = margin(0, 0, 0, 0),
    legend.box.spacing = unit(5, "pt")
  )

ggsave("mic-plot-columns.png", width = 7.5, height = 3.4, unit = "in")



d <- read_excel("mic_data.xlsx", sheet = "F2A",
                col_names = c("strain", "amb", "co2", "growth"),
                skip = 1) %>%
  mutate(is_mic = amb == min(amb[growth < 10]), .by = c(strain, co2),
         is_mic = if_else(amb == 0, FALSE, is_mic)) %>%
  mutate(amb = factor(round(amb, digits = 3)),
         co2 = factor(100 * co2,
                      levels = c("0.04", "1.5", "5.5")),
         strain = factor(
           strain,
           levels = c("R1", "rca1", "efg1", "rca1efg1", "nce103",
                          "rca1nce103", "rca1eNce103", "efg1eNCE103",
                          "ptc2", "rca1ptc2", "rca1RCA1", "efg1EFG1",
                          "nce103NCE103", "S", "B8441"),
           labels = c("R1", "*rca1*&Delta;", "*efg1*&Delta;",
                          "*rca1*&Delta;*efg1*&Delta;", "*nce103*&Delta;",
                          "*rca1*&Delta;*nce103*&Delta;", 
                          "*rca1*&Delta;*eNCE103*",
                          "*efg1*&Delta;*eNCE103*", "*ptc2*&Delta;",
                          "*rca1*&Delta;*ptc2*&Delta;", "*rca1*&Delta;*RCA1*",
                          "*efg1*&Delta;*EFG1*", "*nce103*&Delta;*NCE103*",
                          "S", "AR387"
           ))
  )

d %>%
  ggplot(aes(x = amb, y = co2, fill = growth, stroke = is_mic, color = is_mic)) +
  facet_wrap(.~strain, ncol = 1, strip.position = "left") +
  geom_point(shape = 21, size = 4.75) + 
  geom_text(data = filter(d, is_mic), label = "MIC", color = "#8E8E8E",
            family = "nunito", size = 5.5, size.unit = "pt") +
  annotate(geom = "segment",
           y = 3.6, x = -4.5, xend = 11.5, linewidth = 0.2,
           layout = 2:15) +
  scale_fill_gradient(low = "#FFFFFF", high = "#D08C5E") +
  scale_color_manual(
    values = c("TRUE" = "#CE3769", "FALSE" = "#6C6059")
    # breaks = c(TRUE, FALSE),
    # values = c("#CE3769", "#6C6059")
  ) +
  scale_discrete_manual(
    aesthetics = "stroke",
    values = c("TRUE" = 0.8, "FALSE" = 0.5)
  ) +
  scale_y_discrete(position = "right") +
  coord_cartesian(expand = FALSE, clip = "off",
                  xlim = c(0.5, 11.5),
                  ylim = c(0.5, 3.5)) +
  guides(color = "none", stroke = "none") +
  labs(x = "Amphotericin B (&mu;g ml<sup>-1</sup>)",
       y = "CO<sub>2</sub>",
       fill = "Relative growth (%)") +
  theme(
    text = element_text(family = "nunito"),
    panel.grid = element_blank(),
    panel.background = element_blank(),
    panel.spacing.y = unit(3, "pt"),

    axis.title.y.right = element_markdown(margin = margin(r = -10)),
    axis.title.x = element_markdown(size = 9),
    axis.text.y = element_markdown(color = "black"),
    axis.text.x = element_text(angle = 45, hjust = 1, color = "black",
                               size = 7.5),
    axis.ticks = element_blank(),

    strip.text.y.left = element_markdown(margin = margin(0, 0, 3, 0), angle = 0,
                                         hjust = 1),
    strip.background = element_blank(),
    strip.placement = "outside",

    legend.title = element_text(angle = -90, vjust = 0.2, size = 9),
    legend.text = element_text(size = 8),
    legend.key.width = unit(10, "pt"),
    legend.key.height = unit(14, "pt"),
    legend.position = "right",
    legend.justification.right = "top",
    legend.margin = margin(0, 0, 0, 0),
    legend.box.spacing = unit(5, "pt")
  )

ggsave("mic-plot-rows.png", height = 8.5,  width = 4, unit = "in")
```