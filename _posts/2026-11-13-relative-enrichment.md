---
layout: post
title: "How to indicate gating for a data-rich scatter plot with R's ggplot2 (CC437)"
blurb: "Fun with annotation layers"
author: "PD Schloss"
date: 2026-11-13 09:00
comments: false
youtube: YU-Y21GLUZg
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

* functions: aes, annotate, arrange, coord_cartesian, download.file, element_markdown, element_text, filter, geom_hline, geom_point, geom_polygon, geom_richtext, geom_segment, geom_text, geom_vline, ggplot, ggsave, labs, library, log10, margin, max, min, read_excel, scale_color_manual, scale_x_continuous, scale_y_continuous, select, summarize, theme, theme_classic, tibble


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

data <- read_excel("figure_1.xlsx", sheet = "Fig. 1d") %>%
  select(id = fragment_id,
  active = `log10_enrichment_-BCM`,
  inactive = `log10_enrichment_+BCM`,
  intrinsic = intrinsic_terminator_overlap,
  csss = cssS_AS_fragment)

data %>%
  summarize(xmin = min(inactive), xmax = max(inactive),
            ymin = min(active), ymax = max(active))

csss <- data %>%
  filter(csss)



data %>%
  arrange(intrinsic) %>% 
  # mutate(
  #   quadrant = active > 0 & (10^inactive / 10^active) < 0.45 #inactive two fold lower than active
  # ) %>%
  # ggplot(aes(x = inactive, y = active, color = quadrant)) +
  ggplot(aes(x = inactive, y = active, color = intrinsic)) +
  annotate(
    geom = "polygon", fill = "#F2F3FA",
    x = c(-2, log10(0.45), 2, -2),
    y = c(0, 0, log10(100/0.45), log10(100/0.45)),
  ) +
  geom_vline(xintercept = 0, color = "#C0BBB1") +
  geom_hline(yintercept = 0, color = "#C0BBB1") + 
  geom_point(size = 0.1, show.legend = FALSE) +
  geom_text(data = csss, label = "\u2605", color = "#683696", size = 4) +
  geom_richtext(
    data = tibble(active = 1.25, inactive = 0.55, intrinsic = NA),
    label = "<i>cssS<sup>AS</sup></i>",
    color = "black", label.color = NA, fill = NA, size = 6) +
  annotate(geom = "segment",
           x = 0.55, xend = 0.42, y = 1.18, yend = 1.05) +
  annotate(
    geom = "segment", linetype = "dashed",
    x = c(-2, log10(0.45)),
    xend = c(log10(0.45), 2),
    y = c(0, 0), yend = c(0, log10(100/0.45))
  ) +
  annotate(geom = "text",
           x = -1.15, y = 1.5, label = "Rho-terminated\nfragments (~10,000)",
           color = "#656991", hjust = 0, size = 14, size.unit = "pt",
           lineheight = 1) +
  annotate(geom = "text",
           x = 1.7, y = 0.5, label = "Intrinsic\nterminator",
           color = "#F96259", hjust = 1, size = 14, size.unit = "pt",
           lineheight = 1) +
  annotate(geom = "text",
           x = 1.7, y = -1.0, label = "All fragments\n(n = 161,000)",
           color = "black", hjust = 1, size = 14, size.unit = "pt",
           lineheight = 1) + 
  scale_color_manual(
    breaks = c("FALSE", "TRUE"),
    values = c("#C0BBB1", "#F96259")
  ) +
  scale_x_continuous(
    breaks = c(-1.1, 0, 1),
    labels = c("Below<br>det.", "10<sup>0</sup>", "10<sup>1</sup>")
  ) +
  scale_y_continuous(
    breaks = c(-1.4, -1, 0, 1),
    labels = c("Below<br>det.", "10<sup>-1</sup>", "10<sup>0</sup>", "10<sup>1</sup>")
  ) +
  labs(
    x = "Enrichment (Rho inactive)",
    y = "Enrichment (Rho active)"
  ) +  coord_cartesian(
    expand = FALSE,
    xlim = c(-1.2, 1.7),
    ylim = c(-1.5, 1.7)) + #-1.47  1.50
  theme_classic() + 
  theme(
    axis.text.x = element_markdown(size = 13),
    axis.text.y = element_markdown(size = 13),
    axis.title.x = element_text(size = 14, margin = margin(t = -8)),
    axis.title.y = element_text(size = 14, margin = margin(r = -8))
  )

ggsave("figure_1d.png", width = 5.00, height = 4.82)
```