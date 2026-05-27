---
layout: post
title: "Two methods of creating a jitter dodged slope plot in R with ggplot2 (CC429)"
blurb: "So much better than the original"
author: "PD Schloss"
date: 2026-05-29 09:00
comments: false
youtube: _EPCR09K1Lk
start_hash: 
end_hash: 
---

Pat refactors a difficult to interpret jitter plot from a study that looked at the effects of acarbose and the microbiome on allergic responses. He highlights the importance of having a visualization that reflects the design of the study. This is done by replacing a scatter plot with a plot showing line segments that have arrow heads showing the direction of change of each animal in the study.

* Critique video: https://youtu.be/g8wdteeviao
* Related livestream: https://youtube.com/live/Q9f7JQhl644
* The article discussed: https://www.nature.com/articles/s41564-026-02350-2
* Sign up for a complimentary consultation to help me learn your needs: https://calendly.com/pat-riffomonas/30min
* My newsletter: https://shop.riffomonas.org/youtube 
* Recorded workshops: https://riffomonas.org/workshops/
* If you want to cite this video: https://journals.asm.org/doi/10.1128/mra.01310-22


* packages: tidyverse, readxl, broom, glue, ggplot2, dplyr, purrr

* functions: aes, as.numeric, download.file, element_blank, element_line, element_text, everything, expansion, facet_wrap, factor, geom_jitter, geom_line, geom_path, geom_point, geom_text, ggplot, ggsave, glue, if_else, labs, levels, library, map, margin, mutate, nest, p.adjust, paste, pivot_longer, pivot_wider, position_jitter, read_excel, rep, round, runif, scale_fill_manual, scale_x_continuous, scale_y_continuous, select, separate_wider_delim, set.seed, t.test, theme, theme_classic, tidy, unit, unnest, and wilcox.test


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

```R
library(tidyverse)
library(readxl)
library(broom)
library(glue)

download.file("https://static-content.springer.com/esm/art%3A10.1038%2Fs41564-026-02350-2/MediaObjects/41564_2026_2350_MOESM6_ESM.xlsx",
              "fig_3.xlsx")


replicates <- rep(1:8, 8)
treatment <- rep(c("Ctrl", "Acr", "Abx", "Abx+Acr"), each = 8, times = 2)
time <- rep(c("Before", "After"), each = 32)
col_names <- paste(time, treatment, replicates, sep = "_")

d <- read_excel("fig_3.xlsx", sheet = "Fig. 3d", range = "C5:BN5",
           col_names = col_names) %>%
  mutate(index = 1, .before = everything()) %>%
  pivot_longer(-index, names_to = "col_name", values_to = "shannon") %>%
  separate_wider_delim(cols = col_name, delim = "_", 
                       names = c("time", "treatment", "replicate")) %>%
  select(-index) %>%
  mutate(time = factor(time, levels = c("Before", "After")),
         treatment = factor(treatment, 
                            levels = c("Ctrl", "Acr", "Abx", "Abx+Acr"),
                            labels = c("Control", "Acarbose", "Antibiotics",
                                       "Antibiotics+\nAcarbose")))


p <- d %>%
  pivot_wider(names_from = time, values_from = shannon) %>%
  nest(data = -treatment) %>%
  mutate(test = map(data,
                    ~t.test(.x$After, .x$Before, paired = TRUE) %>% tidy())) %>%
                # ~wilcox.test(.x$After, .x$Before, paired = TRUE) %>% tidy())) %>%
  unnest(test) %>%
  select(treatment, p.value) %>%
  mutate(p.value = p.adjust(p.value, method = "BH"),
         pretty_p = if_else(p.value < 0.001, "P < 0.001",
                            glue("P = {round(p.value, 3)}")))


set.seed(19760620)
jitter <- 0.1
dodge <- 0.2

d %>%
  mutate(x = as.numeric(treatment),
         x = if_else(time == "Before", x - dodge, x + dodge),
         x = x + runif(n(), -jitter, jitter)) %>%
  ggplot(aes(x = x, y = shannon, fill = time,
             group = paste(treatment, replicate))) +
  geom_line(linewidth = 0.3) +
  geom_point(shape = 21) +
  geom_text(data = p, aes(x = as.numeric(treatment), y = 6.3, label = pretty_p),
            size = 7, size.unit = "pt",
            inherit.aes = FALSE) +
  scale_x_continuous(
    breaks = 1:4,
    labels = levels(d$treatment)
  ) +
  scale_y_continuous(
    limits = c(0, 6.5), expand = expansion()
  ) +
  scale_fill_manual(
    values = c(Before = "white", After = "dodgerblue")
  ) +
  labs(
    x = NULL,
    y = "Shannon index",
    fill = NULL
  ) +
  theme_classic() +
  theme(
    axis.text = element_text(size = 7),
    axis.title = element_text(size = 7),
    axis.line = element_line(linewidth = 0.2),
    axis.ticks = element_line(linewidth = 0.2),
    legend.position = "inside",
    legend.position.inside = c(0.2, 0.2),
    legend.text = element_text(size = 7),
    legend.key.size = unit(8, "pt")
  )

ggsave("fig_3d_manual.png", width = 2.5, height = 2.45)




d %>%
  ggplot(aes(x = as.numeric(time), y = shannon, fill = time,
             group = replicate)) +
  geom_path(linewidth = 0.3,
            position = position_jitter(seed = 19760620, width = 0.2)) +
  geom_jitter(shape = 21,
              position = position_jitter(seed = 19760620, width = 0.2)) +
  geom_text(data = p, aes(x = 1.5, y = 6.3, label = pretty_p),
            size = 7, size.unit = "pt",
            inherit.aes = FALSE) +
  facet_wrap(~treatment, nrow = 1, strip.position = "bottom") +
  labs(x = NULL, y = "Shannon index", fill = NULL)  +
  scale_x_continuous(
    breaks = 1.5, labels = "",
    expand = expansion(add = 0.5)
  ) +
  scale_y_continuous(
    limits = c(0, 6.5), expand = expansion()
  ) +
  scale_fill_manual(
    values = c(Before = "white", After = "dodgerblue")
  ) +
  theme_classic() + 
  theme(
    axis.title = element_text(size = 7),
    axis.text.y = element_text(size = 7),
    axis.text.x = element_blank(),
    strip.placement = "outside",
    strip.background = element_blank(),
    strip.text = element_text(margin = margin(), size = 7, vjust = 1),
    axis.line = element_line(linewidth = 0.2),
    axis.ticks = element_line(linewidth = 0.2),
    legend.position = "inside",
    legend.position.inside = c(0.2, 0.2),
    legend.text = element_text(size = 7),
    legend.key.size = unit(8, "pt"),
    panel.spacing.x = unit(0, "pt")
  )


ggsave("fig_3d_facet.png", width = 2.5, height = 2.45)
```