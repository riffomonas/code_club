---
layout: post
title: "Using geom_tile to recreate a genome tiling plot from Nature Microbiology (CC434)"
blurb: "Fun with a gradient legend"
author: "PD Schloss"
date: 2026-11-06 09:00
comments: false
youtube: 
start_hash: 
end_hash: 
---

Pat refactors a plot from a paper recently published in the journal *Nature Microbiology* that depicts the result of a screen using a genome tiling library.

* Critique video: https://youtu.be/D95idif5t80
* Related livestream: https://youtube.com/live/OXij0fqYRws
* The article discussed: https://www.nature.com/articles/s41564-026-02389-1
* Sign up for a complimentary consultation to help me learn your needs: https://calendly.com/pat-riffomonas/30min
* My newsletter: https://shop.riffomonas.org/youtube 
* Recorded workshops: https://riffomonas.org/workshops/
* If you want to cite this video: https://journals.asm.org/doi/10.1128/mra.01310-22


* packages: tidyverse, readxl, ggtext

* functions: aes, annotate, arrange, download.file, element_line, element_text, filter, geom_segment, ggplot, label_comma, labs, library, mutate, n, read_excel, scale_color_gradientn, scale_x_continuous, scale_y_continuous, select, theme, unit


> This work was supported, in part, by a grant from the National Institutes of Health (R25GM166785)

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



```R
library(tidyverse)
library(readxl)
library(ggtext)

download.file(
  "https://static-content.springer.com/esm/art%3A10.1038%2Fs41564-026-02389-1/MediaObjects/41564_2026_2389_MOESM8_ESM.xlsx",
  "figure_1.xlsx"
)

read_excel("figure_1.xlsx", sheet = "Fig. 1d") %>%
  select(id = fragment_id,
         start = fragment_start,
         end = fragment_end,
         strand = fragment_strand,
         enrichment = `log10_enrichment_-BCM`) %>%
  filter(strand == "-") %>%
  filter(end < 3387750 & start > 3386700) %>%
  arrange(start) %>%
  mutate(index = 1:n()) %>%
  ggplot(aes(x = end, xend = start, y = index, color = enrichment)) +
  annotate(geom = "rect", fill = "#ECECF8", 
           xmin = 3387102, xmax = 3387440,
           ymin = -0.1, ymax = 20.5) +
  annotate(geom = "segment",
           x = 3387102, xend = 3387440, y = 0, linewidth = 5,
           color = "#BFC5E5") +
  annotate(geom = "text",
           label = "Known Rho target",
           x = 3387271, y = 0, hjust = 0.5, vjust = 0.5,
           size = 8, size.unit = "pt") +
  geom_segment(linewidth = 2.5) +
  scale_x_continuous(
    breaks = c(3387700, 3387300, 3386900, 3386500),
    labels = scales::label_comma(),
    transform = "reverse") +
  scale_y_continuous(
    limits = c(-0.5, 20.5),
    expand = c(0, 0)
  ) +
  scale_color_gradientn(
    name = "Enrichment (Rho active)",
    colors = c("#736B66", "#A59E95", "white", "#72327F", "#210128"),
    values = c(0, 0.15, 0.5, 0.85, 1),
    limits = c(-1.3, 1.3),
    labels = c(0.1, 1, 10),
    breaks = c(-1, 0, 1)
  ) +
  labs(
    x = "Position (nt)", y = NULL
  ) +
  theme(
    legend.title.position = "right",
    legend.title = element_text(angle = -90, hjust = 0.5),
    legend.key.height = unit(40, "pt"),
    legend.key.width = unit(10, "pt"),
    legend.ticks = element_line(color = "black", linewidth = 0.3),
    legend.ticks.length = unit(c(-3, 0), "pt"),
    
    panel.background = element_blank(),
    panel.grid.major.y = element_blank(),
    panel.grid.minor.y = element_blank(),
    panel.grid.major.x = element_line(color = "gray90", linewidth = 0.2),
    panel.grid.minor.x = element_line(color = "gray90", linewidth = 0.2),
    
    axis.text.x = element_text(color = "black"),
    axis.line.x = element_line(color = "black", linewidth = 0.3),
    axis.ticks.x = element_line(color = "black", linewidth = 0.3),
    axis.text.y = element_blank(),
    axis.ticks.y = element_blank()
  )

ggsave("figure_1e.png", width = 5.00, height = 3.55)
```