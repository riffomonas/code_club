---
layout: post
title: "How to create alternating background colors in R with ggplot2 (CC137)"
blurb: "Recreating a figure showing desire to receive COVID-19 vaccine"
author: "PD Schloss"
date: 2021-08-17 11:00
comments: false
youtube: GYu8kVDxz6A
---

Putting rectangles with alternating colors in the background of a figure can be an attactive way to repalce gridlines. These colored strips can help your audience see what data go together. But how can you do this in R? In this episode of Code Club, Pat will morph an figure he made that was originally creaed by Ipsos to a more stylized one published by chartr. The data depict the percentage of people in 15 countries who would be willing to receive the COVID-19 vaccine as of August and October of 2020.

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

strip_data <- data %>%
  select(country) %>%
  mutate(y_position = 1:nrow(.),
         ymin = y_position - 0.5,
         ymax = y_position + 0.5,
         xmin = 50,
         xmax = 100,
         fill = c(rep(c("a", "b"), length.out=15), "c")
  ) %>%
  pivot_longer(cols=c(xmin, xmax), names_to="min_max", values_to="x")


data %>%
  mutate(index = rev(1:nrow(data))) %>%
  pivot_longer(cols = -c(country, index), names_to=c(".value", "month"),
               names_sep = "_") %>%
  mutate(country = factor(country, levels = rev(data$country))) %>%
  ggplot(aes(x=percent, y=index, color=month, group=index)) +
  geom_ribbon(data=strip_data, aes(x=x, ymin=ymin, ymax=ymax, fill=fill, group=country),
              inherit.aes=FALSE) +
  geom_line(color="#e6e6e6", size=1.75, show.legend = FALSE) +
  geom_point(size=2, show.legend = FALSE) +
  geom_text(aes(label=glue("{percent}%"), x=bump),size=3, show.legend = FALSE) +
  scale_color_manual(name=NULL,
                     breaks=c("august", "october"),
                     values=c("#727272", "#15607a"),
                     labels=c("August", "October")) +
  scale_x_continuous(limits=c(50, 100),
                     breaks=seq(50, 100, by=5),
                     labels=glue("{seq(50, 100, 5)}%"),
                     expand=c(0,0)) +
  scale_y_continuous(limits=c(0.5, 16.5),
                     breaks=c(seq(1, 16, 1), seq(0.5, 16.5, 1)),
                     labels=c(rev(data$country), rep("", 17)),
                     expand=c(0,0)) +
  coord_cartesian(clip="off") +
  labs(x=NULL, y=NULL,
       title="If a vaccine for COVID-19 were available, I would get it",
       caption="<i>Base: 18,526 online adults aged 16-74 across 15 countries</i><br>Source: Ipsos")+
  theme(
    plot.title.position = "plot",
    plot.title = element_text(face="bold", margin= margin(b=20)),
    plot.caption = element_markdown(hjust=0, color="darkgray"),
    plot.caption.position = "plot",
    panel.background = element_blank(),

    axis.text.x = element_text(color="darkgray"),
    axis.ticks.x = element_blank(),
    axis.ticks.y = element_line(color=c(rep(NA, 16), rep("darkgray", 17)), size=0.2),
    axis.line = element_line(color="darkgray", size=0.2),

    panel.grid.major.x = element_blank(),
    panel.grid.major.y = element_blank()
  )

ggsave("august_october_2020_ipsos.tiff", width=6, height=4)
```

### Initial R script
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
