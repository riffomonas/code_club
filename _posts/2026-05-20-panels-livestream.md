---
layout: post
title: "Simplifying a set of 12 panels to better compare the data (CC425)"
blurb: "Less is more"
author: "PD Schloss"
date: 2026-05-20 10:00
comments: false
youtube: fXwsWC2ovZg
start_hash: 
end_hash: 
---

The article discussed: https://www.nature.com/articles/s41586-026-10444-4
Related critique: https://youtu.be/Cmg0HKlhloQ
Code from this livestream: https://riffomonas.org/code_club/2026-05-20-panels-livestream
Sign up for a complimentary consultation to help me learn your needs: https://calendly.com/pat-riffomonas/30min
My newsletter: https://shop.riffomonas.org/youtube 
Recorded workshops: https://riffomonas.org/workshops/
If you want to cite this video: https://journals.asm.org/doi/10.1128/mra.01310-22

packages: tidyverse, readxl, and glue

functions: aes, annotate, arrange, as.numeric, bind_rows, coord_cartesian, download.file, element_blank, element_line, element_text, expansion, facet_grid, factor, filter, geom_errorbar, geom_line, geom_point, ggplot, ggsave, glue, if_else, label_wrap_gen, labeller, labs, library, map, margin, mutate, read_excel, scale_color_manual, scale_y_continuous, select, str_remove, theme, theme_classic, and unit


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

```
library(tidyverse)
library(readxl)
library(glue)

download.file("https://static-content.springer.com/esm/art%3A10.1038%2Fs41586-026-10444-4/MediaObjects/41586_2026_10444_MOESM5_ESM.xlsx",
              "figure_1.xlsx")

d <- glue("Figure 1{letters[9:20]}") %>%
  map(~read_excel("figure_1.xlsx", sheet = .x)) %>%
  bind_rows(.id = "panel") %>%
  select(panel, drug = Injection, genotype = Genotype, time = Time,
         mean = mean_FI, sem = sem_FI) %>%
  mutate(diet = if_else(as.numeric(panel) <= 6, "sd", "hfd"),
         diet = factor(diet,
                       levels = c("sd", "hfd"),
                       labels = c("Active phase:\nStandard diet\nconsumed (g)",
                                  "Inactive phase:\nHigh fat diet\nconsumed (g)")),
         genotype = factor(genotype,
                           levels = c("WT", "S33W"),
                           labels = c("Wild-type", "Humanized (S33W)")),
         drug = if_else(drug == "Saline", "Vehicle", drug),
         drug = factor(drug, 
                       levels = c("Vehicle", "Liraglutide",
                                  "Danuglipron", "Orfo"),
                       labels = c("Vehicle", "Liraglutide",
                                  "Danuglipron", "Orforglipron")),
         time = str_remove(time, "h.*r") %>% as.numeric(),
         grouping_set = factor(paste0(drug, panel),
                               levels = c("Vehicle1", "Vehicle2", "Vehicle3",
                                          "Vehicle4", "Vehicle5", "Vehicle6",
                                          "Vehicle7", "Vehicle8", "Vehicle9",
                                          "Vehicle10", "Vehicle11", "Vehicle12",
                                          "Liraglutide1", "Liraglutide2",
                                          "Liraglutide7", "Liraglutide8",
                                          "Danuglipron3", "Danuglipron4",
                                          "Danuglipron9", "Danuglipron10", 
                                          "Orforglipron5", "Orforglipron6",
                                          "Orforglipron11", "Orforglipron12")
         )) %>%
  arrange(grouping_set) 


d %>%
  ggplot(aes(x = time, y = mean, color = drug, group = grouping_set,
             ymin = mean - sem, ymax = mean + sem)) +
  geom_line(data = filter(d, drug == "Vehicle"), linewidth = 0.3) +
  geom_point(data = filter(d, drug == "Vehicle")) +
  geom_errorbar(data = filter(d, drug == "Vehicle"),
                width = 0.1, linewidth = 0.3) +
  geom_line(data = filter(d, drug != "Vehicle"), linewidth = 0.3) +
  geom_errorbar(data = filter(d, drug != "Vehicle"),
                width = 0.1, linewidth = 0.3) +
  geom_point(data = filter(d, drug != "Vehicle")) +
  facet_grid(diet ~ genotype, switch = "y",
             labeller = labeller(diet = label_wrap_gen(width = 20),
                                 genotype = label_wrap_gen(width = 10))) +
  scale_color_manual(
    values = c(Vehicle = "gray", Orforglipron = "#B51B8F",
               Liraglutide = "#25A6E2", Danuglipron = "#E92C2D")) +
  scale_y_continuous(expand = expansion()) +
  coord_cartesian(ylim = c(0, 2.4), clip = "off") +
  labs(
    color = NULL,
    x = "Time (h)",
    y = NULL
  ) +
  annotate(
    geom = "text",
    x = c(2, 4), y = c(0.2, 0.3), label = c("***", "***"),
    color = "#25A6E2", layout = 1,
    size = 8, size.unit = "pt"
  ) +
  annotate(
    geom = "text",
    x = c(2, 4, 2, 4, 1, 2, 4), y = c(0.2, 0.45, 0.1, 0.35,-0.05, 0.0, 0.25),
    label = c("*", "***", "***", "***", "*", "***", "***"),
    color = c("#25A6E2", "#25A6E2", "#E92C2D", "#E92C2D",
              "#B51B8F","#B51B8F","#B51B8F"), layout = 2,
    size = 8, size.unit = "pt"
  ) +
  annotate(
    geom = "text",
    x = c(1, 2, 4), y = c(0.35, 0.45, 0.55), label = c("*", "**", "***"),
    color = "#25A6E2", layout = 3,
    size = 8, size.unit = "pt"
  ) +
  annotate(
    geom = "text",
    x = c(0.9, 2, 4, 1, 2, 4, 1.1, 2, 4),
    y = c(0.1, 0.35, 0.4, 0.0, 0.25, 0.3, 0.1, 0.15, 0.2),
    label = c("*", "**", "***", "***", "***", "***", "*", "**", "**"),
    color = c("#25A6E2", "#25A6E2", "#25A6E2",
              "#E92C2D", "#E92C2D","#E92C2D",
              "#B51B8F","#B51B8F","#B51B8F"), layout = 4,
    size = 8, size.unit = "pt"
  ) +
  theme_classic() +
  theme(
    strip.placement = "outside",
    strip.text = element_text(size = 8, margin = margin()),
    strip.clip = "off",
    strip.background = element_blank(),
    axis.title.x = element_text(size = 8),
    axis.text = element_text(size = 7),
    axis.text.y = element_text(margin = margin(l = 3, r = 1)),
    axis.line = element_line(linewidth = 0.2),
    axis.ticks = element_line(linewidth = 0.2),
    legend.key.size = unit(10, "pt"),
    legend.key.spacing = unit(3, "pt"),
    legend.text = element_text(size = 7),
    legend.box.spacing = unit(3, "pt")
  )  
  
# ggsave("figure_1.png", width = 7.5, height = 2.87) #original aspect ratio
ggsave("figure_1.png", width = 4.5, height = 2.87)
```