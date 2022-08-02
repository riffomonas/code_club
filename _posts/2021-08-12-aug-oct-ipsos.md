---
layout: post
title: "Recreating a labelled dumbbell chart in R showing people's openness to the COVID-19 vaccine (CC136)"
blurb: "Get vaccinated!"
author: "PD Schloss"
date: 2021-08-12 11:00
comments: false
youtube: qaksmQabMUI
---

In December 2020, Ipsos released a report that contained a labelled dumbbell chart (also called a barbell chart or plot) that showed the percentage of people who said they would likely receive the COVID-19 vaccine once it was available. I thought this was an interesting figure, in part, because of how it depicted paired data. In this episode we'll recreate the original plot in R so that in future episodes we can modify it and experiment with other ways of representing the data.

Pat uses functions from the tidyverse including functions from the `ggplot2`, `dplyr`, `ggtext` and `glue` packages in RStudio.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

### Final R script

```R
library(tidyverse)
library(glue)
library(ggtext)

data <- read_csv("august_october_2020.csv") %>%
  rename(country = X.1,
         percent_august = "Total Agree - August 2020",
         percent_october = "Total Agree - October 2020") %>%
  mutate(bump_august = if_else(percent_august < percent_october,
                               percent_august - 2,
                               percent_august + 2),
         bump_october = if_else(percent_august < percent_october,
                               percent_october + 2,
                               percent_october - 2))

main_plot <- data %>%
  pivot_longer(cols = -country, names_to=c(".value", "month"),
               names_sep = "_") %>%
  mutate(country = factor(country, levels = rev(data$country))) %>%
  ggplot(aes(x=percent, y=country, color=month)) +
  geom_line(color="#e6e6e6", size=1.75, show.legend = FALSE) +
  geom_point(size=2, show.legend = FALSE) +
  geom_text(aes(label=glue("{percent}%"), x=bump),size=3, show.legend = FALSE) +
  scale_color_manual(name=NULL,
                     breaks=c("august", "october"),
                     values=c("#727272", "#15607a"),
                     labels=c("August", "October")) +
  scale_x_continuous(limits=c(50, 100),
                     breaks=seq(50, 100, by=5),
                     labels=glue("{seq(50, 100, 5)}%")) +
  labs(x=NULL, y=NULL,
       title="If a vaccine for COVID-19 were available, I would get it",
       caption="<i>Base: 18,526 online adults aged 16-74 across 15 countries</i><br>Source: Ipsos")+
  theme(
    plot.title.position = "plot",
    plot.title = element_text(face="bold", margin= margin(b=20)),
    plot.caption = element_markdown(hjust=0, color="darkgray"),
    plot.caption.position = "plot",
    panel.background = element_blank(),
    axis.ticks = element_blank(),
    axis.text.x = element_text(color="darkgray"),
    panel.grid.major.x = element_line(color="gray", size=0.1),
    panel.grid.major.y = element_line(color="gray", size=0.1, linetype="dotted")
  )

total <- data %>%
  filter(country == "Total") %>%
  pivot_longer(cols = -country, names_to=c(".value", "month"),
             names_sep = "_") %>%
  mutate(pretty = if_else(month == "august",
                          "Total Agree -<br>August 2020",
                          "Total Agree -<br>October 2020"),
         align = if_else(month == "august", 0, 1))

main_plot +
  coord_cartesian(clip="off") +
  geom_textbox(data=total,
               aes(x=percent, y =country, color=month, label=pretty, hjust=align),
               size=2,
               box.color=NA,
               width=NULL,
               vjust=-0.5,
               box.padding=margin(0,0,0,0),
               fill=NA,
               show.legend=FALSE)

ggsave("august_october_2020_ipsos.tiff", width=6, height=4)
```

### Data

```
X.1,Total Agree - August 2020,Total Agree - October 2020
Total,77,73
India,87,87
China,97,85
South Korea,84,83
Brazil,88,81
Australia,88,79
United Kingdom,85,79
Mexico,75,78
Canada,76,76
Germany,67,69
Japan,75,69
South Africa,64,68
Italy,67,65
Spain,72,64
United States,67,64
France,59,54
```
