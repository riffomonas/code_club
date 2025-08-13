---
layout: post
title: "Using ggplot2 to compare people's preference for different card games (CC365)"
blurb: "A horizontal stacked barplot"
author: "PD Schloss"
date: 2025-08-13 9:00
comments: false
youtube: N9RUVfTNUF8
start_hash: 
end_hash: 
---

Pat recreates a horizontal stacked barplot from YouGov that shows people's preferences for different card games using the ggplot2 R package. The plot was generated using functions from the tidyverse including the dplyr, ggplot2, showtext, and ggtext packages. The functions he uses from these packages include aes, arrange, as.character, coord_cartesian, cumsum, element_blank, element_text, element_textbox_simple, font_add_google, geom_col, geom_text, ggplot, ggsave, labs, length, library, margin, mutate, nest, pivot_longer, pivot_wider, position_stack, scale_fill_manual, showtext_auto, showtext_opts, theme, tribble, unit, and unnest. The newsletter describing this visualization at a 30,000 ft view can be [found here](https://shop.riffomonas.org/posts/visualizing-how-americans-feel-about-different-card-games). You can find the original article describing the data [here](https://today.yougov.com/entertainment/articles/45795-how-americans-feel-about-30-card-games). The data are availble as a [PDF here](https://d3nkl3psvxxpe9.cloudfront.net/documents/crosstabs_Card_Games.pdf). If you have a figure that you would like to see me discuss in a future newsletter and episode of Code Club, email me at pat@riffomonas.org!


Want more practice on the concepts covered in Code Club? You can sign up for my weekly newsletter at https://shop.riffomonas.org/youtube to get practice problems, tips, and insights. If you're interested in purchasing a video workshop be sure to check out https://riffomonas.org/workshops/

Support Riffomonas by becoming a Patreon member!
https://www.patreon.com/riffomonas

You can also find complete tutorials for learning R with the tidyverse using...
Microbial ecology data: https://www.riffomonas.org/minimalR/
General data: https://www.riffomonas.org/generalR/

If you want to cite this video, please consider citing https://journals.asm.org/doi/10.1128/mra.01310-22



```R
library(tidyverse)
library(ggtext)
library(showtext)

font_add_google("Libre Franklin", "franklin", bold.wt = 900, regular.wt = 500)
showtext_opts(dpi = 300)
showtext_auto()

# 1. Get data from YouGov
#    * data clean up
#    * formatting data to "work":
#      * re-scale the numbers
#      * order of sentiments
#      * order of loving
# 2. Create horizontal stacked bar plot
# 3. Implement the color scheme for bars / format/place legend
# 4. Inserting numbers into bars
#    * remove values under 4%
#    * fix x-axis placement
#    * color the numbers accordingly
# 5. Titles

# 6. Other formatting
x_nudge <- 2.25

tribble(~game, ~"love", ~"like", ~"dislike", ~"hate", ~"unsure",
        "Blackjack", 26, 57, 10, 1, 4,
        "Bridge", 19, 44, 18, 4, 15,
        "Bullsh*t", 31, 51, 9, 1, 8,
        "Canasta", 20, 55, 13, 1, 12,
        "Crazy Eights", 14, 66, 8, 1, 11,
        "Cribbage", 26, 49, 10, 4, 11,
        "Euchre", 30, 50, 11, 3, 6,
        "Gin Rummy", 24, 53, 12, 2, 8,
        "Go Fish", 14, 60, 17, 2, 7,
        # "Hand and Foot", 33, 48, 9, 2, 7,
        "Hearts", 20, 62, 10, 1, 8,
        # "Kemps", 36, 31, 25, 0, 8,
        "Kings in the Corner", 15, 60, 7, 1, 16,
        "Old Maid", 11, 58, 18, 1, 12,
        # "Omaha", 21, 62, 9, 2, 7,
        "Pinochle", 34, 35, 14, 3, 13,
        # "Pitch", 33, 40, 14, 2, 11,
        # "Pitty Pat", 26, 43, 12, 11, 7,
        "Poker", 22, 53, 17, 2, 7,
        "Pyramid", 22, 53, 15, 0, 9,
        "Seven Card Stud", 20, 57, 10, 2, 12,
        "Solitaire", 34, 55, 8, 2, 2,
        "Spades", 30, 56, 8, 2, 3,
        "Spit or Speed", 38, 41, 12, 3, 6,
        "Spoons", 28, 54, 9, 2, 7,
        "Texas Hold’em", 28, 53, 12, 3, 4,
        # "Thirty-one", 21, 45, 13, 4, 17,
        "War", 20, 57, 13, 1, 8#,
        # "Whist", 33, 48, 9, 5, 5,
        # "Zwickern", 21, 52, 19, 2, 5
  ) %>%
  nest(label = -game) %>%
  mutate(percent = map(label, \(x) {100 * x / sum(x)} )) %>%
  unnest(c(label, percent), names_sep = "_") %>%
  arrange(percent_love, desc(game)) %>% 
  mutate(game = factor(game, levels = game)) %>%
  pivot_longer(-game, names_to = c("type", "sentiment"), values_to = "value",
               names_sep = "_") %>%
  pivot_wider(names_from = type, values_from = value) %>%
  mutate(sentiment = factor(sentiment,
                            levels = c("love", "like", "unsure", "dislike",
                                       "hate"),
                            labels = c("Love it", "Like it", "Not sure",
                                       "Dislike it", "Hate it")),
         label = if_else(label < 4, "", as.character(label))) %>%
  arrange(sentiment) %>%
  mutate(label_x = c(x_nudge,
                     x_nudge + cumsum(percent)[1:(length(percent) - 1)]),
                    .by = game) %>%
  ggplot(aes(x = percent, y = game, fill = sentiment)) +
  geom_col(position = position_stack(reverse = TRUE),
           width = 0.85) +
  # geom_text(aes(x = percent, label = label),
  #           position = position_stack(reverse = TRUE, vjust = 0.1)) +
  geom_text(aes(x = label_x, label = label, color = sentiment),
            family = "franklin", size = 8, size.unit = "pt",
            show.legend = FALSE) +
  scale_fill_manual(name = NULL,
                    values = c("#A029FF", "#C57FFF", "#CCD1DB",
                               "#FF8D80", "#FF412C")) +
  scale_color_manual(name = NULL,
                     values = c("white", "white", "black",
                                "#000000", "#FFFFFF")) +
  coord_cartesian(expand = FALSE) +
  labs(
    title = "Most Americans <span style='color:#A029FF;'>love</span> or <span style='color:#C57FFF;'>like</span> card games they've played",
    subtitle = "How do you feel about playing the following card game? (% of U.S. adult citizens who say they have ever played each card game)",
    caption = "Note: Graphic only includes card games which at least 100 respondents said they had ever played before.",
    x = NULL,
    y = NULL
  ) +
  # theme_void() +
  theme(
    text = element_text(family = "franklin"),
    legend.position = "top",
    legend.justification.top = "left",
    legend.key.size = unit(11, "pt"),
    legend.text = element_text(size = 8, margin = margin(l = 1, r = 2)),
    legend.margin = margin(t = 10, b = -3, l = 10),
    plot.title.position = "plot",
    plot.caption.position = "plot",
    plot.caption = element_text(hjust = 0, face = "italic", size = 8,
                                margin = margin(t = 12)),
    plot.title = element_textbox_simple(face = "bold", size = 15),
    plot.subtitle = element_textbox_simple(size = 8.5, lineheight = 1.4,
                                           margin = margin(t = 8)),
    plot.margin = margin(t = 4, r = 0, b = 4, l = 5),
    panel.grid = element_blank(),
    panel.background = element_blank(),
    axis.ticks = element_blank(),
    axis.text.x = element_blank(),
    axis.text.y = element_text(size = 8, margin = margin(l = 3))
  )

ggsave("card-games.png", width = 6, height = 6.5)
```