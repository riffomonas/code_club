---
layout: post
title: "Finding alternatives to plots with two y axes (CC441)"
blurb: "Six different attempts"
author: "PD Schloss"
date: 2026-11-25 09:00
comments: false
youtube: LxK_dWanjxc
start_hash: 
end_hash: 
---

The article discussed: https://www.nature.com/articles/s41564-026-02458-5

Related critique: https://youtu.be/_b8V8h6AxEk

Sign up for a complimentary consultation to help me learn your needs: https://calendly.com/pat-riffomonas/30min
My newsletter: https://shop.riffomonas.org/youtube 
Recorded workshops: https://riffomonas.org/workshops/
If you want to cite this video: https://journals.asm.org/doi/10.1128/mra.01310-22

* packages: tidyverse, readxl,patchwork, ggtext

* functions: aes, annotate, case_when, coord_cartesian, download.file, element_blank, element_markdown, element_text, facet_wrap, factor, filter, geom_line, geom_path, geom_point, geom_rect, geom_segment, geom_smooth, geom_text, ggplot, ggsave, inner_join, labs, library, margin, max, mean, min, mutate, pivot_longer, plot_layout, read_excel, scale_color_manual, scale_linewidth_manual, scale_x_date, scale_y_continuous, sec_axis, select, summarize, theme, theme_classic, year, ymd

This work was supported, in part, by a grant from the National Institutes of Health (R25GM166785)

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



```R
library(tidyverse)
library(readxl)
library(patchwork)
library(ggtext)

# obtaining and formatting data -----

download.file(
  "https://static-content.springer.com/esm/art%3A10.1038%2Fs41564-026-02458-5/MediaObjects/41564_2026_2458_MOESM5_ESM.xlsx",
  "fig_1.xlsx")

d <- read_excel("fig_1.xlsx", sheet = "Figure1D") %>%
  select(date, pm2p5, hurs, disease) %>%
  mutate(date = ymd(date))

# 1 panel, 3 lines, without scaling -----

d %>%
  pivot_longer(cols = -date) %>%
  ggplot(aes(x = date, y = value, color = name)) +
  geom_line()
  
ggsave("one_panel_wo_scaling.png", width = 5, height = 2.1)



# 1 panel, 3 lines, with scaling -----

pre_vax_means <- d %>%
  filter(date < ymd("2009-01-01")) %>%
  summarize(pm2p5 = mean(pm2p5),
            hurs = mean(hurs),
            disease = mean(disease)) %>%
  pivot_longer(cols = c(pm2p5, hurs, disease), values_to = "mean")

d %>%
  pivot_longer(-date) %>%
  inner_join(., pre_vax_means, by = "name") %>%
  mutate(percent_change = 100 * ((value / mean) - 1)) %>%
  ggplot(aes(x = date, y = percent_change, color = name)) +
  geom_line()


ggsave("one_panel_w_scaling.png", width = 5, height = 2.1)



# 3 facets, 1 line per panel -----

d %>%
  pivot_longer(-date) %>%
  mutate(name = factor(name,
                       levels = c("hurs", "disease", "pm2p5"),
                       labels = c("Relative<br>Humidity (%)",
                                  "Case Counts",
                                  "Particulate Matter<br>(PM<sub>2.5</sub> &mu; g m<sup>-3</sup>)"))) %>% #april through september
  ggplot(aes(x = date, y = value)) +
  annotate(
    geom = "rect", layout = 1,
    xmin = ymd(paste0(2005:2023, "-04-01")),
    xmax = ymd(paste0(2005:2023, "-10-01")),
    ymin = 35,
    ymax = 75, fill = "lightblue", alpha = 0.4
  ) +
  annotate(
    geom = "rect", layout = 2,
    xmin = ymd(paste0(2005:2023, "-04-01")),
    xmax = ymd(paste0(2005:2023, "-10-01")),
    ymin = 70,
    ymax = 590, fill = "lightblue", alpha = 0.4
  ) +
  annotate(
    geom = "rect", layout = 3,
    xmin = ymd(paste0(2005:2023, "-04-01")),
    xmax = ymd(paste0(2005:2023, "-10-01")),
    ymin = 6,
    ymax = 48, fill = "lightblue", alpha = 0.4
  ) +
  geom_line(show.legend = FALSE) + 
  facet_wrap(~name, ncol = 1, scales = "free_y", strip.position = "left") +
  labs(y = NULL, x = "Date (months)") +
  scale_x_date(
    limits = ymd(c("2004-09-01", "2025-04-01")), expand = c(0, 0)
  ) +
  scale_y_continuous(expand = c(0, 0)) +
  theme_classic() +
  theme(
    plot.margin = margin(4, 10, 3, 3),
    strip.placement = "outside",
    strip.background = element_blank(),
    strip.text.y.left = element_markdown(margin = margin(r = -4)),
    strip.clip = "off"
  )

ggsave("three_panels.png", width = 4, height = 5)



# 2 panels, double y-axis -----

scaling <- d %>%
  summarize(min_pm = min(pm2p5),
            scale_pm = (max(disease) - min(disease))/(max(pm2p5) - min(pm2p5)),
            
            min_h = min(hurs),
            scale_h = (max(disease) - min(disease))/(max(hurs) - min(hurs)),
            
            min_disease = min(disease))

pm <- d %>%
  select(-hurs) %>%
  mutate(tr_pm2p5 = (pm2p5 - scaling$min_pm) * scaling$scale_pm + scaling$min_disease) %>%
  select(-pm2p5) %>%
  pivot_longer(-date) %>%
  ggplot(aes(x = date, y = value, color = name, linewidth = name)) +
  geom_line(show.legend = FALSE) +
  scale_color_manual(
    values = c(disease = "black", tr_pm2p5 = "red")
  ) +
  scale_y_continuous(
    sec.axis = sec_axis(
      transform = ~.,
      name = "PM",
      breaks = (c(10, 20, 30, 40) - scaling$min_pm) * scaling$scale_pm + scaling$min_disease,
      labels = c(10, 20, 30, 40)
    )
  ) +
  scale_linewidth_manual(
    values = c(disease = 0.5, tr_pm2p5 = 0.25)
    ) +
  labs(
    y = "Case counts"
  ) +
  theme(
    axis.title.y.right = element_text(color = "red")
  )


h <- d %>%
  select(-pm2p5) %>%
  mutate(tr_hurs = (hurs - scaling$min_h) * scaling$scale_h + scaling$min_disease) %>%
  select(-hurs) %>%
  pivot_longer(-date) %>%
  ggplot(aes(x = date, y = value, color = name, linewidth = name)) +
  geom_line(show.legend = FALSE) +
  scale_color_manual(
    values = c(disease = "black", tr_hurs = "blue")
  ) +
  scale_linewidth_manual(
    values = c(disease = 0.5, tr_hurs = 0.25)
  ) +
  scale_y_continuous(
    sec.axis = sec_axis(
      transform = ~.,
      name = "Humidity",
      breaks = (c(40, 50, 60, 70) - scaling$min_h) * scaling$scale_h + scaling$min_disease,
      labels = c(40, 50, 60, 70)
    )
  ) +
  labs(
    y = "Case counts"
  ) +
  theme(
    axis.title.y.right = element_text(color = "blue")
  )

pm / h +
  plot_layout(axes = "collect")

ggsave("double_y_axis.png", width = 5, height = 2.1)



# 1 panel, triple y-axis -----

d %>%
  mutate(
    tr_pm2p5 = (pm2p5 - scaling$min_pm) * scaling$scale_pm + scaling$min_disease,
    tr_hurs = (hurs - scaling$min_h) * scaling$scale_h + scaling$min_disease
    ) %>%
  select(-pm2p5, -hurs) %>%
  pivot_longer(-date) %>%
  ggplot(aes(x = date, y = value, color = name, linewidth = name)) +
  geom_line(show.legend = FALSE) +
  scale_color_manual(
    values = c(disease = "black", tr_pm2p5 = "red", tr_hurs = "blue")
  ) +
  scale_linewidth_manual(
    values = c(disease = 0.5, tr_pm2p5 = 0.25, tr_hurs = 0.25)
  ) +
  scale_y_continuous(
    sec.axis = sec_axis(
      transform = ~.,
      name = "PM",
      breaks = (c(10, 20, 30, 40) - scaling$min_pm) * scaling$scale_pm + scaling$min_disease,
      labels = c(10, 20, 30, 40)
    )
  ) +
  coord_cartesian(clip = "off",
                  xlim = ymd(c("2004-06-01", "2024-06-01")),
                  ylim = c(50, 590),
                  expand = FALSE) +
  labs(
    y = "Case counts"
  ) +
  theme_classic() +
  theme(
    axis.title.y.right = element_text(color = "red"),
    plot.margin = margin(t = 10, b = 10, l = 10, r = 50)
  ) +
  annotate(
    geom = "text", angle = -90,
    x = ymd("2030-01-01"), y = 325, label = "Humidity", color = "blue"
  ) +
  annotate(
    geom = "text", hjust = 0, size = 9, size.unit = "pt",
    x = ymd("2028-01-01"),
    y = (c(40, 50, 60, 70) - scaling$min_h) * scaling$scale_h + scaling$min_disease,
    label = c(40, 50, 60, 70)
  ) +
  annotate(
    geom = "segment",
    x = ymd("2027-08-01"), 
    xend = ymd("2027-11-01"), 
    y = (c(40, 50, 60, 70) - scaling$min_h) * scaling$scale_h + scaling$min_disease,
  ) +
  annotate(
    geom = "segment",
    x = ymd("2027-08-01"),
    y = 50, yend = 590
  )

ggsave("triple_y_axis.png", width = 5, height = 2.1)



# connected scatterplots -----

d %>%
  pivot_longer(cols = c(pm2p5, hurs)) %>%
  mutate(period = case_when(year(date) < 2009 ~ "pre_vax",
                            year(date) > 2019 ~ "covid",
                            TRUE ~ "with_vax")) %>%
  ggplot(aes(x = value, y = disease, color = period)) +
  # geom_path() +
  geom_point() +
  facet_wrap(~name, nrow = 1, scales = "free_x",
             strip.position = "bottom") +
  theme(strip.placement = "outside") +
  labs(x = NULL) +
  geom_smooth(method = "lm",color = "black")
  
ggsave("connected_scatterplots.png", width = 5, height = 2.1)
```