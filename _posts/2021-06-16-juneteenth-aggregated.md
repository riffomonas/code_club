---
layout: post
title: "Juneteenth 2021: a data visualization of lynchings using R's ggplot2 package (CC116)"
blurb: "Using R skills for good"
author: "PD Schloss"
date: 2021-06-16 11:00
comments: false
youtube: y6OhR6MW3Mw
---

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Data

You can request the data I used in this episode from the [CSDE Lynching Database](http://lynching.csde.washington.edu/#/home).

## Visuals

<p><img src="assets/images/lynchings_per_year.tiff" width="75%"></p>

<p><img src="assets/images/lynchings_per_state_per_year.tiff" width="75%"></p>


## Code

These are the two R scripts I generated in this episode

```R
library(tidyverse)
library(glue)
library(ggtext)


lynchings_per_year <- read_csv("Weblist_IDs.csv") %>%
  rename_all(tolower) %>%
  filter(victimsrace == "Black") %>%
  count(year)

total_lynchings <- lynchings_per_year %>%
  summarize(N = sum(n)) %>%
  mutate(N = format(N, big.mark=",")) %>%
  pull(N)

peak_year <- lynchings_per_year %>%
  top_n(n, n=1) %>%
  pull(year)

year_range <- lynchings_per_year %>%
  summarize(early = min(year), late = max(year)) %>%
  mutate(range = glue("{early} and {late}")) %>%
  pull(range)

lynchings_per_year %>%
  ggplot(aes(x=year, y=n)) +
  geom_line() +
  labs(x="Year", y="Number of Lynchings",
       title=glue("Between {year_range} there were {total_lynchings} lynchings
                  with the peak occuring in {peak_year}"),
       caption = "Data provided by CSDE Lynchings Database") +
  theme_classic() +
  theme(plot.title = element_textbox_simple(size=18, face="bold",
                                            margin = margin(t=10, 0, b=15, 0)),
        plot.title.position = "plot",
        plot.caption.position = "plot",
        plot.caption = element_text(face="italic"))


ggsave("lynchings_per_year.tiff", width=5, height=5)
```

```R
library(tidyverse)
library(glue)
library(ggtext)

state_lookup <- tibble(abb = state.abb,
       name = state.name)

lynchings_per_state_per_year <- read_csv("Weblist_IDs.csv") %>%
  rename_all(tolower) %>%
  filter(victimsrace == "Black") %>%
  count(year, lynchstate) %>%
  inner_join(., state_lookup, by=c("lynchstate" = "abb"))

lynchings_per_state <- lynchings_per_state_per_year %>%
  group_by(name) %>%
  summarize(N = sum(n)) %>%
  mutate(state = glue("{name} ({N})"))

peak_state <- lynchings_per_state %>%
  top_n(n=1, N) %>%
  pull(name)

year_range <- lynchings_per_state_per_year %>%
  summarize(early = min(year), late = max(year)) %>%
  mutate(range = glue("{early} and {late}")) %>%
  pull(range)

lynchings_per_state_per_year %>%
  inner_join(., lynchings_per_state, by="name") %>%
  mutate(state = fct_reorder(state, N)) %>%
  ggplot(aes(x=year, fill=n, y=state)) +
  geom_tile() +
  scale_fill_gradient(name = "Number of\nLynchings",
                      low="#FFFFFF", high = "#FF0000",
                      limits=c(0, NA)) +
  labs(x="Year", y=NULL,
       title=glue("Between {year_range} {peak_state} had the most lynchings"),
       caption = "Data provided by CSDE Lynchings Database") +
  theme_classic() +
  theme(axis.line = element_blank(),
        axis.ticks = element_blank(),
        plot.title = element_textbox_simple(size=18, face="bold",
                                          margin = margin(t=5, 0, b=10, 0)),
        plot.title.position = "plot",
        plot.caption.position = "plot",
        plot.caption = element_text(face="italic"))

ggsave("lynchings_per_state_per_year.tiff", width=5, height=5)
```

## Installations

* [See video](https://www.youtube.com/watch?v=D6CunpqF04E) for instructions on getting set up
* [R](https://r-project.org)
* [RStudio](https://rstudio.com)
* [Raw data](https://github.com/riffomonas/raw_data/releases/latest)
