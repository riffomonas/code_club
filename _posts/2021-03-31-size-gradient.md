---
layout: post
title: "Scaling the size of points in an ordiantion by diversity"
blurb: "A bubble chart for microbial ecology"
author: "PD Schloss"
date: 2021-03-31 11:30
comments: false
youtube: J6U-eKLxj2M
---

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

This is where we started before the episode

```R
library(tidyverse)
library(readxl)
library(ggtext)
library(glue)

pcoa <- read_tsv(file="raw_data/schubert.braycurtis.pcoa.axes")
metadata <- read_excel(path="raw_data/schubert.metadata.xlsx", na="NA")

metadata_pcoa <- inner_join(metadata, pcoa, by=c('sample_id'='group')) %>%
  mutate(disease_stat = factor(disease_stat,
                               levels=c("DiarrhealControl",
                                        "Case",
                                        "NonDiarrhealControl")
                               )
         )

lim <- max(abs(c(pcoa$axis1, pcoa$axis2)))# set the limits to be square, but
limits <- c(-1*lim, lim)                  # include all of the points

explained <- read_tsv("raw_data/schubert.braycurtis.pcoa.loadings")
explained1 <- explained %>% filter(axis==1) %>% pull(loading) %>% round(1)
explained2 <- explained %>% filter(axis==2) %>% pull(loading) %>% round(1)


alpha_diversity <- read_tsv("raw_data/schubert.groups.ave-std.summary") %>%
  filter(method == "ave") %>%
  select(-label, -method)

metadata_pcoa_alpha <- inner_join(metadata_pcoa, alpha_diversity,
                                  by=c("sample_id"="group"))

healthy_color <- "#BEBEBE"
diarrhea_color <- "#0000FF"
case_color <- "#FF0000"



ggplot(metadata_pcoa_alpha,
       aes(x=axis1, y=axis2, color=disease_stat, fill=disease_stat)) +
  stat_ellipse(geom="polygon",type="norm", level=0.75, alpha=0.05,
               show.legend=F) +
  geom_point(show.legend=F) +
  coord_cartesian(xlim=limits, ylim=limits) +
  labs(title=glue("<span style='color: {healthy_color}'><strong>Healthy \\
                  individuals</strong></span> have a different microbiota \\
                  from<br><span style='color: {diarrhea_color}'><strong>those \\
                  with diarrhea</strong></span> and those with diarrhea \\
                  who<br>are <span style='color: {case_color}'><strong>\\
                  positive for *C. difficile*</strong></span>"),
       x=glue("PCo Axis 1 ({explained1}%)"),
       y=glue("PCo Axis 2 ({explained2}%)"),
       caption=glue("All pairwise comparisons were significant using ADONIS \\
                    at 0.05 using<br>Benjimani-Hochberg correction for \\
                    multiple comparisons")) +
  scale_color_manual(name=NULL,
                     breaks=c("NonDiarrhealControl", "DiarrhealControl", "Case"),
                     values=c(healthy_color, diarrhea_color, case_color),
                     labels=c("Healthy", "Diarrhea", "*C. difficile* positive")
                     )+
  scale_fill_manual(name=NULL,
                     breaks=c("NonDiarrhealControl", "DiarrhealControl", "Case"),
                    values=c(healthy_color, diarrhea_color, case_color),
                    labels=c("Healthy", "Diarrhea", "*C. difficile* positive")
  )+
  theme_classic() +
  theme(
    legend.key.size = unit(0.25, "cm"),
    legend.position = c(1.0, 0.95),
    legend.background = element_rect(fill="white", color="black"),
    legend.margin = margin(t=-2, r=3, b=3, l=3),
    legend.text = element_markdown(),
    plot.margin = margin(t=0.5, l=0.5, r=2, b=0.5, unit="lines"),
    plot.title.position = "plot",
    plot.title = element_markdown(margin = margin(b=1, unit="lines")),
    axis.title.y = element_text(hjust = 1),
    axis.title.x = element_text(hjust = 0),
    plot.caption = element_markdown(hjust=0),
    plot.caption.position = "plot"
  )

ggsave("schubert_pcoa.tiff", width=4.5, height=4.5)
```

## Installations

* [See video](https://www.youtube.com/watch?v=D6CunpqF04E) for instructions on getting set up
* [R](https://r-project.org)
* [RStudio](https://rstudio.com)
* [Raw data](https://github.com/riffomonas/raw_data/releases/latest)
