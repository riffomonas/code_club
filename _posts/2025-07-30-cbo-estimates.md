---
layout: post
title: "Using ggplot2 to desmontrate the regressive nature of Trump's spending bill (CC364)"
blurb: "Making a bar plot more sophisticated"
author: "PD Schloss"
date: 2025-07-23 9:00
comments: false
youtube: -tx7p68I0m4
start_hash: 
end_hash: 
---

Pat recreates a barplot from the New York Times showing the regressive impacts of Trump's spending bill on various income groups in the United States using the ggplot2 R package. The plot was generated using functions from the tidyverse including the dplyr, ggplot2, readxl, showtext, ggtext, and glue packages. The functions he uses from these packages include aes, as.numeric, case_when, download.file, element_rect, element_text, element_textbox_simple, factor, font_add_google, function, geom_col, geom_hline, geom_text, ggplot, glue, if_else, labs, library, margin, mutate, read_excel, scale_color_manual, scale_fill_manual, scale_x_discrete, select, showtext_auto, showtext_opts, str_remove, theme, and theme_void. The newsletter describing this visualization at a 30,000 ft view can be [found here](https://shop.riffomonas.org/posts/how-would-you-make-a-labelled-bar-plot-with-positive-and-negative-values). You can find the original article describing the data [here for free](https://archive.is/jG4bM). The data are [availble from the CBO]https://archive.is/ksh2e. If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```R
library(tidyverse)
library(showtext)
library(ggtext)
library(readxl)
library(glue)

font_add_google("Libre Franklin", "franklin", regular.wt = 500)
font_add_google("Libre Franklin", "franklin_light")
showtext_opts(dpi = 300)
showtext_auto()

# 1. Get the data! Link in bottom right corner of plot to the CBO
#     * In MS Excel format
#     * Read in with {readxl}
# 2. Do some data clean up to get in tidy format to make bar plot
#     * create a variable to indicate positive or negative net effect
#     * create bar plot with geom_col()
#     * add text with geom_text()
#     * stylize the percent to have the percent sign and positive/negative sign
#     * color label by direction of effect
#     * match the colors and remove the legend
#
# glue("+{net_effect}%")
# paste0("+", net_effect, "%")
#
# 3. Axis work
#     * remove x and y axis lines, text, ticks
#     * remove gridlines and background
#     * add zero line with geom_hline
#     * add decile information to top of negative bars, bottom of positive bars
# 4. Add titles
#     * Need to stylize various elements

nd_to_range <- function(x){
  
  decile_number <- as.numeric(str_remove(x, "\\w\\w$"))
  upper <- 10 * decile_number
  lower <- 10 * (decile_number - 1)
  
  glue("{lower}th-\n{upper}th")
  
}

decile_to_text <- function(x){
  
  case_when(x == "Lowest" ~ "Bottom\n10%",
            x == "Highest" ~ "Top\n10%",
            TRUE ~ nd_to_range(x))

}

data_url <- "https://www.cbo.gov/system/files/2025-06/61387-Data.xlsx"
download.file(data_url, "cbo-data.xlsx")

nudge <- 0.2

read_excel("cbo-data.xlsx", sheet = "Figure 2", range = "A8:F18") %>%
  select(decile = "Household \r\nincome decile",
         net_effect = "Net effect") %>%
  mutate(direction = if_else(net_effect < 0, "negative", "positive"),
         decile = decile_to_text(decile),
         decile = factor(decile, levels = decile),
         decile_y = if_else(net_effect < 0, 0.45, -0.45),
         label_y = if_else(net_effect < 0,
                           net_effect - nudge, net_effect + nudge),
         label = if_else(net_effect < 0,
                         glue("{net_effect}%"),
                         glue("+{net_effect}%"))) %>%
  ggplot(aes(x = decile, y = net_effect, fill = direction,
             label = label)) +
  geom_col(width = 0.8) +
  geom_text(aes(y = label_y, color = direction),
            family = "franklin", size = 9, size.unit = "pt") +
  geom_text(aes(y = decile_y, label = decile),
            color = "gray50", lineheight = 0.9,
            size = 8, size.unit = "pt", family = "franklin_light") +
  geom_hline(yintercept = 0, linewidth = 0.3) +
  scale_fill_manual(breaks = c("negative", "positive"),
                    values = c("#D35400", "#59A497")) +
  scale_color_manual(breaks = c("negative", "positive"),
                    values = c("#D35400", "#59A497")) +
  scale_x_discrete(expand = expansion(0, 0.48)) +
  labs(
    title = "How the Bill Would Affect Households at Different Income Ranks",
    subtitle = "Estimated annual average change in resources between 2026-34",
    caption = "Note: Estimated annual average effect of the House version of the One Big Beautiful Bill Act on after-tax income. Groups are based on income adjusted for household size.&nbsp; &nbsp; &bull; &nbsp; &nbsp; Source: Congressional Budget Office"
  ) +
  theme_void() +
  theme(
    text = element_text(family = "franklin"),
    legend.position = "none",
    plot.background = element_rect(fill = "white", color = NA),
    plot.title = element_text(face = "bold", size = 11,
                              margin = margin(l = -10)),
    plot.subtitle = element_text(size = 9.5, color = "gray40",
                                 margin = margin(l = -10, t = 8, b = 12)),
    plot.caption = element_textbox_simple(color = "gray40", size = 8,
                                          margin = margin(l = -10, t = 15),
                                          lineheight = 1.4),
    plot.margin = margin(t = 8, l = 18, r = 10, b = 6)
  )```
