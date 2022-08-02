---
layout: post
title: "Reusing R code to repeat an analysis for a new dataset"
blurb: "Rerunning old Code"
author: "PD Schloss"
date: 2021-03-22 11:30
comments: false
youtube: ZJgbCIMVfzM
---

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

This is where we started before the episode

`baxter_nmds.R`

```R
library(tidyverse)
library(readxl)
library(ggtext)

source("schubert_adonis.R")

nmds <- read_tsv(file="raw_data/schubert.braycurtis.nmds_2d.axes")
metadata <- read_excel(path="raw_data/schubert.metadata.xlsx")
metadata_nmds <- inner_join(metadata, nmds, by=c('sample_id'='group')) %>%
  mutate(disease_stat = factor(disease_stat,
                               levels=c("DiarrhealControl",
                                        "Case",
                                        "NonDiarrhealControl")
                               )
         )

my_legend <- tibble(x = c(-0.85, -0.7, 0.52),
                    y = c(0.75, -0.7, 0.83),
                    color = c("Case", "NonDiarrhealControl", "DiarrhealControl"),
                    label = c("<strong>*C. difficile*<br>positive</strong>", "<strong>Healthy</strong>", "<strong>Diarrhea</strong>"))

ggplot(metadata_nmds,
       aes(x=axis1, y=axis2, color=disease_stat, fill=disease_stat)) +
  stat_ellipse(geom="polygon",type="norm", level=0.75, alpha=0.2, show.legend=F) +
  geom_point(show.legend=FALSE) +
  # geom_richtext(data=my_legend,
  #           aes(x=x, y=y, label=label, color=color),
  #           inherit.aes=FALSE, show.legend = FALSE, hjust=0, fill=NA, label.color=NA) +
  coord_cartesian(xlim=c(-0.8, 0.8), ylim=c(-0.8, 0.8)) +
  labs(title="<span style='color: #999999'><strong>Healthy individuals</strong></span> have a different microbiota from<br><span style='color: #0000FF'><strong>those with diarrhea</strong></span> and those with diarrhea who<br>are <span style='color: #FF0000'><strong>positive for *C. difficile*</strong></span>",
       x="NMDS Axis 1",
       y="NMDS Axis 2",
       caption="All pairwise comparisons were significant using ADONIS at 0.05 using\nBenjimani-Hochberg correction for multiple comparisons") +
  scale_color_manual(name=NULL,
                     breaks=c("NonDiarrhealControl", "DiarrhealControl", "Case"),
                     values=c("gray", "blue", "red"),
                     labels=c("Healthy", "Diarrhea", "*C. difficile* positive")
                     )+
  scale_fill_manual(name=NULL,
                     breaks=c("NonDiarrhealControl", "DiarrhealControl", "Case"),
                     values=c("lightgray", "dodgerblue", "pink"),
                     labels=c("Healthy", "Diarrhea", "*C. difficile* positive")
  )+
  theme_classic() +
  theme(
    legend.key.size = unit(0.25, "cm"),
    legend.position = c(1.0, 0.95),
    legend.background = element_rect(fill="white", color="black"),
    legend.margin = margin(t=-2, r=3, b=3, l=3),
    legend.text = element_markdown(),
    plot.margin = margin(l=1, r=4, unit="lines"),
    plot.title.position = "plot",
    plot.title = element_markdown(margin = margin(b=1, unit="lines")),
    axis.title.y = element_text(hjust = 1),
    axis.title.x = element_text(hjust = 0),
    plot.caption = element_text(hjust=0),
    plot.caption.position = "plot"
  )

ggsave("schubert_nmds.tiff", width=5, height=4)
```


`baxter_adonis.R`

```R
library(tidyverse)
library(readxl)
library(vegan)

set.seed(19760620)
permutations <- 1000

metadata <- read_excel(path="raw_data/schubert.metadata.xlsx")

distance <- read_tsv("raw_data/schubert.braycurtis.dist",
                     skip=1, col_names=FALSE)

colnames(distance) <- c("sample_id", distance$X1)

meta_distance <- inner_join(metadata, distance, by="sample_id")

all_dist <- meta_distance %>%
  select(all_of(.[["sample_id"]])) %>%
  as.dist()

all_test <- adonis(all_dist~disease_stat,
                   data=meta_distance,
                   permutations = permutations)

all_p <- all_test[["aov.tab"]][["Pr(>F)"]][1]

meta_distance %>% count(disease_stat)

pairwise_p <- numeric()

# Case vs DiarrhealControl
case_diarrhea <- meta_distance %>%
  filter(disease_stat == "Case" | disease_stat == "DiarrhealControl")

case_diarrhea_dist <- case_diarrhea %>%
  select(all_of(.[["sample_id"]])) %>%
  as.dist()

case_diarrhea_test <- adonis(case_diarrhea_dist~disease_stat,
                             data=case_diarrhea,
                             permutations=permutations)

pairwise_p["case_diarrhea"] <- case_diarrhea_test[["aov.tab"]][["Pr(>F)"]][1]



# Case vs NonDiarrhealControl
case_health <- meta_distance %>%
  filter(disease_stat == "Case" | disease_stat == "NonDiarrhealControl")

case_health_dist <- case_health %>%
  select(all_of(.[["sample_id"]])) %>%
  as.dist()

case_health_test <- adonis(case_health_dist~disease_stat,
                             data=case_health,
                             permutations=permutations)

pairwise_p["case_health"] <- case_health_test[["aov.tab"]][["Pr(>F)"]][1]



# DiarrhealControl vs NonDiarrhealControl
diarrhea_health <- meta_distance %>%
  filter(disease_stat == "DiarrhealControl" | disease_stat == "NonDiarrhealControl")

diarrhea_health_dist <- diarrhea_health %>%
  select(all_of(.[["sample_id"]])) %>%
  as.dist()

diarrhea_health_test <- adonis(diarrhea_health_dist~disease_stat,
                           data=diarrhea_health,
                           permutations=permutations)

pairwise_p["diarrhea_health"] <- diarrhea_health_test[["aov.tab"]][["Pr(>F)"]][1]


stopifnot(all(p.adjust(pairwise_p, method="BH") < 0.05))
```


## Installations

* [See video](https://www.youtube.com/watch?v=D6CunpqF04E) for instructions on getting set up
* [R](https://r-project.org)
* [RStudio](https://rstudio.com)
* [Raw data](https://github.com/riffomonas/raw_data/releases/latest)
