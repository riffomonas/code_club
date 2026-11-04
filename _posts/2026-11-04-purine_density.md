---
layout: post
title: "Recreating figures with a lot of data points with R's tidyverse (CC433)"
blurb: "Creating a scatter plot with marginal histograms"
author: "PD Schloss"
date: 2026-11-04 09:00
comments: false
youtube: OXij0fqYRws
start_hash: 
end_hash: 
---

Pat refactors a plot from a paper recently published in the journal *Nature Microbiology* to compare and contrast the benefits of summarizing versus showing all the data.

* Critique video: https://youtu.be/D95idif5t80
* Related livestream: https://youtube.com/live/OXij0fqYRws
* The article discussed: https://www.nature.com/articles/s41564-026-02389-1
* Sign up for a complimentary consultation to help me learn your needs: https://calendly.com/pat-riffomonas/30min
* My newsletter: https://shop.riffomonas.org/youtube 
* Recorded workshops: https://riffomonas.org/workshops/
* If you want to cite this video: https://journals.asm.org/doi/10.1128/mra.01310-22


* packages: tidyverse, readxl, patchwork, ggtext, glue

* functions: aes, download.file, element_blank, element_line, element_markdown, facet_wrap, geom_abline, geom_density, geom_hex, geom_histogram, geom_point, geom_rug, geom_text, ggplot, ggsave, glue, labs, library, margin, pivot_longer, plot_layout, plot_spacer, read_excel, scale_x_continuous, scale_y_continuous, select, theme, theme_classic, tibble


> This work was supported, in part, by a grant from the National Institutes of Health (R25GM166785)

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



```R
library(tidyverse)
library(readxl)
library(patchwork)
library(ggtext)
library(glue)

download.file(
  "https://static-content.springer.com/esm/art%3A10.1038%2Fs41564-026-02389-1/MediaObjects/41564_2026_2389_MOESM8_ESM.xlsx",
  "figure_1.xlsx")

# 1-d histograms
read_excel("figure_1.xlsx", sheet = "Fig. 1b") %>%
  select(
    id = fragment_id,
    pre = `log10_read_count_-BCM_pre`,
    post = `log10_read_count_-BCM_post`
  ) %>%
  pivot_longer(-id) %>%
  ggplot(aes(x = value)) +
  geom_histogram() +
  facet_wrap(~name, ncol = 1)


# 2-d histograms
read_excel("figure_1.xlsx", sheet = "Fig. 1b") %>%
  select(
    id = fragment_id,
    pre = `log10_read_count_-BCM_pre`,
    post = `log10_read_count_-BCM_post`
  ) %>%
  ggplot(aes(x = pre, y = post)) +
  geom_point(size = 0.01) +
  geom_rug(linewidth = 0.05, alpha = 0.2, color = "gray") +
  theme_classic()
#  geom_hex()


ggsave("figure_1b.png", width = 5.00, height = 3.95)


data <- read_excel("figure_1.xlsx", sheet = "Fig. 1b") %>%
  select(
    id = fragment_id,
    pre = `log10_read_count_-BCM_pre`,
    post = `log10_read_count_-BCM_post`
  ) 

remove_background <- theme(
  panel.grid = element_blank(),
  panel.background = element_blank()
)

scatter <- data %>%
  ggplot(aes(x = pre, y = post)) +
  geom_abline(slope = 1, intercept = -2:2,
              linewidth = 0.2, linetype = "dashed", color = "gray40") +
  geom_point(size = 0.01, alpha = 0.2, color = "gray")  + 
  geom_text(data = tibble(
    pre = c(-4, -4, -4, -4.15, -5.15),
    post = c(-5.95, -4.95, -3.95, -3.05, -3.05),
    label = c(0.01, 0.1, 1, 10, 100)),
    aes(label = label), hjust = 1, size = 9, size.unit = "pt",
    color = "gray40") +
  scale_x_continuous(limits = c(-4, -6), expand = c(0, 0),
                     breaks = c(-4, -5, -6),
                     labels = glue("10<sup>{c(-4, -5, -6)}</sup>")) +
  scale_y_continuous(limits = c(-3, -6.3), expand = c(0, 0),
                     breaks = c(-3, -4, -5, -6),
                     labels = glue("10<sup>{c(-3, -4, -5, -6)}</sup>")) +
  labs(x = "Frequency before selection",
       y = "Frequency after selection") +
  theme(axis.text.x = element_markdown(color = "black"),
        axis.text.y = element_markdown(color = "black"),
        axis.ticks = element_line(linewidth = 0.2)) +
  remove_background
  

y_hist <- data %>%
  ggplot(aes(y = post)) +
  geom_histogram(binwidth = 0.1) +
 # geom_density() +
  scale_x_continuous(limits = c(0, 18000), expand = c(0, 0)) +
  scale_y_continuous(limits = c(-3, -6.3), expand = c(0, 0),
                     breaks = c(-3, -4, -5, -6),
                     labels = glue("10<sup>{c(-3, -4, -5, -6)}</sup>")) +
  labs(y = "Frequency after selection") +
  theme(axis.ticks = element_line(linewidth = 0.2),
        axis.ticks.x = element_blank(),
        axis.text.x = element_blank(),
        axis.title.x = element_blank(),
        axis.text.y = element_markdown(color = "black")) +
  remove_background


x_hist <- data %>%
  ggplot(aes(x = pre)) +
 # geom_density() 
  geom_histogram(binwidth = 0.1) +
  scale_y_continuous(transform = "reverse", limits = c(0, 18000),
                     expand = c(0, 0)) +
  scale_x_continuous(limits = c(-4, -6), expand = c(0, 0),
                     breaks = c(-4, -5, -6),
                     labels = glue("10<sup>{c(-4, -5, -6)}</sup>")) +
  labs(x = "Frequency before selection") +
  theme(axis.ticks = element_line(linewidth = 0.2),
        axis.ticks.y = element_blank(),
        axis.text.y = element_blank(),
        axis.title.y = element_blank(),
        axis.text.x = element_markdown(color = "black")) +
  remove_background


scatter + y_hist + x_hist + plot_spacer() +
  plot_layout(axes = "collect", ncol = 2,
              widths = c(1, 0.1), heights = c(1, 0.1)) #&
 # theme(plot.margin = margin(3, 1, 1, 1))

ggsave("figure_1b.png", width = 5.00, height = 6.00)
```