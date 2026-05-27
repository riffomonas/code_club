---
layout: post
title: "Representing paired pre/post data in an ordination with arrows using R and ggplot2 (CC428)"
blurb: "Adding a sense of direction"
author: "PD Schloss"
date: 2026-05-28 09:00
comments: false
youtube: GOa_KjF4hM8
start_hash: 
end_hash: 
---

Pat refactors a difficult to interpret ordination diagram taken from a study that looked at the effects of acarbose and the microbiome on allergic responses. He highlights the importance of having a visualization that reflects the design of the study. This is done by replacing a scatter plot with a plot showing line segments that have arrow heads showing the direction of change of each animal in the study.

* Critique video: https://youtu.be/g8wdteeviao
* Related livestream: https://youtube.com/live/Q9f7JQhl644
* The article discussed: https://www.nature.com/articles/s41564-026-02350-2
* Code from this video: https://riffomonas.org/code_club/2026-05-28-ordination
* Sign up for a complimentary consultation to help me learn your needs: https://calendly.com/pat-riffomonas/30min
* My newsletter: https://shop.riffomonas.org/youtube 
* Recorded workshops: https://riffomonas.org/workshops/
* If you want to cite this video: https://journals.asm.org/doi/10.1128/mra.01310-22


* packages: tidyverse, readxl, dplyr, ggplot2

* functions: aes, arrange, arrow, download.file, element_blank, element_line, element_text, factor, geom_path, ggplot, ggsave, labs, library, mutate, paste, read_excel, rename, rename_all, rep, scale_color_manual, theme, theme_classic, unit


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

```
library(tidyverse)
library(readxl)

download.file("https://static-content.springer.com/esm/art%3A10.1038%2Fs41564-026-02350-2/MediaObjects/41564_2026_2350_MOESM6_ESM.xlsx",
              "fig_3.xlsx")

read_excel("fig_3.xlsx", sheet = "Fig. 3e", range = "C3:E67") %>%
  rename_all(tolower) %>%
  rename(treatment = sample) %>%
  mutate(time = rep(c("Before", "After"), each = 32),
         time = factor(time, levels = c("Before", "After")),
         replicate = paste(treatment, rep(1:8, times = 8)),
         treatment = factor(treatment,
                            levels = c("Ctrl", "Acr", "Abx", "Abx+Acr"))) %>%
  arrange(time) %>%
  ggplot(aes(x = pco1, y = pco2, color = treatment, shape = time,
             group = replicate)) +
  geom_path(linewidth = 0.4,
            arrow = arrow(type = "closed", length = unit(4, "pt"))) +
  scale_color_manual(
    values = c("Ctrl" = "#0700FE", "Acr" = "#FC0504", 
               "Abx" = "#FFA400", "Abx+Acr" = "#9D23EE"),
    labels = c("Ctrl" = "Control", "Acr" = "Acarbose", 
               "Abx" = "Antibiotics", "Abx+Acr" = "Antibiotics+\nAcarbose")
  ) +
  labs(
    x = "PCo 1 (47%)", y = "PCo 2 (22.8%)",
    color = NULL
  ) +
  theme_classic() +
  theme(
    axis.text = element_text(size = 6),
    axis.title = element_text(size = 7),
    axis.line = element_line(linewidth = 0.3),
    axis.ticks = element_line(linewidth = 0.3),
    legend.text = element_text(size = 6, margin = margin(l = 3),
                               lineheight = 0.7),
    legend.key.height = unit(9, "pt"),
    legend.key.width = unit(10, "pt"),
    legend.position = "inside",
    legend.position.inside = c(0.85, 0.85),
    legend.background = element_blank(),
    legend.key.justification = "top"
  )

ggsave("fig_3e.png", width = 2.5, height = 1.765)
```