---
layout: post
title: "Juneteenth 2021: Creating a data based movie in R with gganimate of lynchings (CC118)"
blurb: "Creating a movie to 'say' victims' names"
author: "PD Schloss"
date: 2021-06-21 11:00
comments: false
youtube: OOpci_5BaE8
---

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Data

You can request the data I used in this episode from the [CSDE Lynching Database](http://lynching.csde.washington.edu/#/home).

## The video

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/Xg7Els_N8_c" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

This is the R script that I generated in this episode

```R
library(tidyverse)
library(ggtext)
library(gganimate)
library(glue)
library(lubridate)

set.seed(18650619)

state_lookup <- tibble(abb = state.abb,
                       state = state.name)

lynchings <- read_csv("Weblist_IDs.csv") %>%
  rename_all(tolower) %>%
  filter(victimsrace == "Black") %>%
  inner_join(., state_lookup, by=c("lynchstate" = "abb")) %>%
  mutate(county = tolower(lynchcounty),
         state = tolower(state)) %>%
  select(state, county, name, victimssex, victimsage, methodofdeath, day, month,  year) %>%
  mutate(county = str_replace_all(county, "\\.", ""),
         county = case_when(
           state == "arkansas" & county == "probably white" ~ "white",
           state == "florida" & county == "dade" ~ "miami-dade",
           state == "florida" & county == "desoto" ~ "de soto",
           state == "georgia" & county == "campbell" ~ "fulton",
           state == "georgia" & county =="dekalb" ~ "de kalb",
           state == "georgia" & county == "unidentified southwest georgia county" ~
             "decatur",
           state == "georgia" & county == "wilkerson" ~ "wilkinson",
           state == "louisiana" & county == "arcadia" ~ "bienville",
           state == "louisiana" & county == "desoto" ~ "de soto",
           state == "louisiana" & county == "la fourche" ~ "lafourche",
           state == "mississippi" & county == "desoto" ~ "de soto",
           state == "virginia" & county == "alexandria" ~ "richmond",
           state == "virginia" & county == "princess anne" ~ "virginia beach",
           state == "virginia" & county == "dinwiddlie" ~ "dinwiddie",
           state == "virginia" & county == "warwick" ~ "newport news",
           TRUE ~ county
         )
  ) %>%
    mutate(victimssex = if_else(victimssex == "Unknown", NA_character_, victimssex),
           victimssex = tolower(victimssex)) %>%
    mutate(victimsage = tolower(victimsage),
           victimsage = str_replace(victimsage, '�', ''),
           victimsage = str_replace(victimsage, "^1$", "1 year old"),
           victimsage = str_replace(victimsage, "^(\\d*)$", "\\1 years old"),
           victimsage = str_replace(victimsage, "^(\\d*-\\d*)$", "\\1 years old"),
           victimsage = str_replace(victimsage, "^< ?(\\d*)$", "less than \\1 years old"),
           victimsage = str_replace(victimsage, "^20s$", "20-30 years old"),
           victimsage = str_replace(victimsage, "^about (\\d*)$", "about \\1 years old"),
           victimsage = str_replace(victimsage, "^aged$", "elderly"),
           victimsage = str_replace(victimsage, "^old man$", "elderly"),
           victimsage = str_replace(victimsage, "^old$", "elderly"),
           victimsage = str_replace(victimsage, "^boy$", "young"),
           victimsage = str_replace(victimsage, "^child$", "young"),
           victimsage = str_replace(victimsage, "^youth$", "young"),
           victimsage = str_replace(victimsage, "middled", "middle"),
           victimsage = str_replace(victimsage, "12 months", "12 months old"),
           victimsage = str_replace(victimsage, "late 30s", "35-39 years old"),
           victimsage = na_if(victimsage, "unknown")
    )%>%
    mutate(methodofdeath = str_replace(methodofdeath, "�", " "),
           methodofdeath = str_replace(methodofdeath, ";", ","),
           methodofdeath = str_replace(methodofdeath, " - ", ", "),
           methodofdeath = str_replace(methodofdeath, "-", ", "),
           methodofdeath = str_replace(methodofdeath, "\\s+", " "),
           methodofdeath = str_replace(methodofdeath, " ,", ","),
           methodofdeath = str_replace(methodofdeath, "Hang$", "Hanged"),
           methodofdeath = case_when(
             is.na(methodofdeath) ~ "Lynched",
             methodofdeath == "Not reported" ~ "Lynched",
             methodofdeath == "Unknown" ~ "Lynched",
             methodofdeath == "Unreported" ~ "Lynched",
             methodofdeath == "Unreproted" ~ "Lynched",
             TRUE ~ methodofdeath),
           methodofdeath = str_to_sentence(methodofdeath)
    ) %>%
    mutate(name = str_replace(name, '�', ''),
           name = str_replace(name, '�', ''),
           name = str_replace(name, "^\\s+", ""),
           name = str_replace(name, "^Unnamed.*", "Unnamed"),
           name = str_replace(name, "^Unknown.*", "Unnamed")
    ) %>%
    mutate(day = if_else(day < 1 | day > 31, NA_real_, day),
           month = if_else(month < 1 | month > 12, NA_real_, month),
           day = if_else(is.na(month), NA_real_, day),
           date = case_when(
             is.na(month) ~ glue("{year}"),
             is.na(day) ~ glue("{month(month, abb=FALSE, label=TRUE)} {year}"),
             TRUE ~ glue("{month(month, abb=FALSE, label=TRUE)} {day}, {year}"))
           ) %>%
    mutate(last_lines = glue("<br>{methodofdeath}<br>{date}"),
      caption = case_when(is.na(victimssex) ~ glue("**{name}**, {victimsage}{last_lines}"),
                          is.na(victimsage) ~ glue("**{name}**, {victimssex}{last_lines}"),
                          TRUE ~ glue("**{name}**, {victimssex} {victimsage}{last_lines}")
                          )
      ) %>% select(state, county, caption)


# **Name**, sex and age
# Method of death
# Date
state_county <- map_data("county") %>%
  select(region, subregion) %>%
  distinct()

anti_join(lynchings, state_county,
          by=c("county" = "subregion", "state" = "region"))

county_map <- lynchings %>%
  distinct(state) %>%
  inner_join(., map_data("county"), by=c("state" = "region"))

state_map <- lynchings %>%
  distinct(state) %>%
  inner_join(., map_data("state"), by=c("state" = "region"))


lynchings_map_data <- lynchings %>%
  mutate(n = 1) %>%
  slice_sample(prop=1) %>%
  mutate(order = 1:nrow(.)) %>%
  mutate(caption = fct_reorder(caption, order)) %>%
  right_join(., county_map,
             by=c("state" = "state", "county" = "subregion"))

caption_data <-  lynchings_map_data %>%
  select(caption) %>%
  distinct(caption) %>%
  drop_na(caption)

static <- lynchings_map_data %>%
  drop_na(n) %>%
  ggplot(aes(x=long, y=lat, fill=n, group=group)) +
  geom_polygon(color="black", size=0.1) +
  geom_polygon(data=county_map, aes(x=long, y=lat, group=group),
               fill=NA, color="black", size=0.1) +
  geom_polygon(data=state_map, aes(x=long, y=lat, group=group),
               fill=NA, color="black", size=0.3) +
  geom_textbox(data = caption_data,
               aes(label=caption), x=-95.5, y=41, hjust =0, vjust=1,
               fill = NA, box.color=NA,
               width = unit(3, "in"),
               inherit.aes = FALSE) +
  scale_fill_gradient(name="Number of\nLynchings",
                      low="#FFFFFF", high = "#FF0000",
                      limits=c(0, NA)) +
  coord_quickmap() +
  theme_void() +
  theme(
    plot.title = element_textbox_simple(face="bold", size=18,
                                        margin = margin(5, 0, 10, 5)),
    legend.position = "none"
  )

#ggsave("lynchings_map.tiff", height=4.5, width =8)

animation <- static + transition_manual(caption)

animate(animation,
        height=4.5, width =8, units = "in", res = 300,
        fps = 1, nframes = nrow(caption_data),
        renderer = av_renderer("juneteenth_map_movie.mp4")
        )
```

## Installations

* [See video](https://www.youtube.com/watch?v=D6CunpqF04E) for instructions on getting set up
* [R](https://r-project.org)
* [RStudio](https://rstudio.com)
* [Raw data](https://github.com/riffomonas/raw_data/releases/latest)
