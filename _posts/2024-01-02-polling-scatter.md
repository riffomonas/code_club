---
layout: post
title: "Visualizing polling bias from the 2024 US presidential election in R with ggplot2 (CC329)"
blurb: "Too many aesthetics!"
author: "PD Schloss"
date: 2025-01-02 8:00
comments: false
youtube: xn_kKP16u28
start_hash: 
end_hash: 
---

Pat creates a visualization showing the polling bias from the 2024 US presidential election using ggplot2 in R as a scatter plot with the x, y, color, fill, size, and alpha aesthetics to highlight the difference in polling and actual vote count, number of electoral votes, number of polls, age of the polls, and whether the data were from a battle ground state. This is his attempt to improve upon a colored barbell plot made by Nate Silver. To pull off this visualization, Pat uses the ggplot2, ggtext, and glue R packages. In addition to geom_point, he uses the geom_abline, annotate, scale_fill_gradient, scale_color_manual, scale_x_continuous, scale_y_continuous, labs, coord_equal, theme_classic, and theme functions. He also used the read_csv, mutate, select, glue, and drop_na functions from the readr, dplyr, and glue packages. Nate Silver's newsletter can be found at https://www.natesilver.net/p/hopium-comes-at-a-high-price. Pat's newsletter describing how he would go about generating the figure can be [found here](https://shop.riffomonas.org/posts/visualizing-bias-in-polling-data-with-a-dumbbell-plot).

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

```R
library(tidyverse)
library(glue)
library(ggtext)

battle_ground_states <- c("Arizona", "North Carolina", "Nevada", "Georgia",
                          "Pennsylvania", "Wisconsin", "Michigan")
  
margins <- read_csv("2024_presidential_data.csv",
         col_types = cols(.default = col_double(),
                          state = col_character(),
                          last_poll = col_character())) %>%
  mutate(polling = trump_poll - harris_poll,
         actual = trump_vote - harris_vote,
         last_poll = mdy(last_poll),
         age = as.numeric(as.Date("2024-11-04") - last_poll), 
         battle_ground = state %in% battle_ground_states,
         trump_better = actual > polling) %>%
  select(state, polling, age, actual, n_polls, last_poll,
         battle_ground, electoral_votes, trump_better) %>%
  drop_na()
  
  n_trump_better <- sum(margins$trump_better)
  w_polling_data <- length(margins$trump_better)
  
margins %>%
  ggplot(aes(x = actual, y = polling, size = n_polls, alpha = -age,
             fill = electoral_votes, color = battle_ground)) +
  geom_abline(slope = 1, intercept = 0, color = "gray", linetype = "dashed") +
  geom_point(shape = 21, show.legend = FALSE) +
  annotate("text", x = c(-30, 10), y = c(45, -30), hjust = 0, color = "gray",
           label = c("Harris outperformed polls", "Trump outperformed polls")) +
  scale_fill_gradient(low = "white", high = "darkgreen",
                      limits = c(0, 20),
                      breaks = c(0, 5, 10, 15, 20),
                      labels = c(0, 5, 10, 15, ">20"),
                      na.value = "darkgreen") +
  scale_color_manual(breaks = c(FALSE, TRUE),
                     values = c("black", "gold")) +
  scale_x_continuous(breaks = seq(-30, 45, 15),
                     labels = c("<span style='color:blue'>+30</span>", "<span style='color:blue'>+15</span>", "<span style='color:black'>0</span>", "<span style='color:red'>+15</span>", "<span style='color:red'>+30</span>", "<span style='color:red'>+45</span>")) +
  scale_y_continuous(breaks = seq(-30, 45, 15),
                     labels = c("<span style='color:blue'>+30</span>", "<span style='color:blue'>+15</span>", "<span style='color:black'>0</span>", "<span style='color:red'>+15</span>", "<span style='color:red'>+30</span>", "<span style='color:red'>+45</span>")) +
  #  scale_alpha_gradient()
  labs(x = "Actual vote difference",
       y = "Polling difference",
       title = glue("In {n_trump_better} of {w_polling_data} contests, <span style='color:red'>Trump</span> outperformed the polls relative to <span style='color:blue'>Harris</span>"),
       subtitle = "Each plotting symbol is colored by the <span style='color:darkgreen'>**number of electoral votes**</span> with the <span style='color:gold'>**battle ground states highlighted in a golden ring**</span>. States with more polling data have large points and those with older data are more transparent") +
  coord_equal() +
  theme_classic() +
  theme(
    plot.title.position = "plot",
    plot.title = element_textbox_simple(face = "bold"),
    plot.subtitle = element_textbox_simple(size = 8, margin = margin(t = 5, b = 10)),
    axis.text = element_markdown(face = "bold")
  )

ggsave("polling_bias.png", width = 5, height = 5)
```

The data from `2024_presidential_data.csv`...

```csv
state,harris_poll,trump_poll,n_polls,last_poll,harris_vote,trump_vote,electoral_votes
National,48.4,47.2,25,11/4/24,48.4,49.9,0
Alabama,NA,NA,NA,NA,34.2,64.8,9
Alaska,43,51,1,10/27/24,41.4,54.5,3
Arizona,46.8,48.5,17,11/4/24,46.7,52.2,11
Arkansas,40,55,1,9/13/24,33.5,64.2,6
California,59,34.2,5,11/4/24,58.5,38.3,54
Colorado,52.5,42.5,2,11/3/24,54.2,43.2,10
Connecticut,57,41,1,9/23/24,56.4,41.9,7
Delaware,55,36.5,2,9/27/24,56.6,41.9,3
District of Columbia,NA,NA,NA,NA,92.5,6.6,3
Florida,44.9,51.1,8,11/4/24,43,56.1,30
Georgia,47.5,48.7,15,11/4/24,48.4,50.7,16
Hawaii,NA,NA,NA,NA,60.6,37.5,4
Idaho,NA,NA,NA,NA,30.4,66.9,4
Illinois,NA,NA,NA,NA,54.8,43.8,19
Indiana,39.5,56,2,9/29/24,39.7,58.6,11
Iowa,45.3,50,4,11/4/24,42.7,56,6
Kansas,43,48,1,10/28/24,41,57.2,6
Kentucky,NA,NA,NA,NA,33.9,64.6,8
Louisiana,NA,NA,NA,NA,38.2,60.2,8
Maine,50.3,41.7,3,11/3/24,52.4,45.5,2
Maine-1,58,35.3,3,11/3/24,59.5,38.1,1
Maine-2,42.7,48.7,3,11/3/24,44.2,53.2,1
Maryland,60.2,33,5,11/3/24,63,34.4,10
Massachusetts,59.5,31.8,4,11/3/24,61.3,36.5,11
Michigan,48.6,46.8,23,11/4/24,48.3,49.7,15
Minnesota,49.8,43.6,5,11/4/24,46.9,51.1,10
Mississippi,NA,NA,NA,NA,37.3,61.6,6
Missouri,39,54,1,11/4/24,40.1,58.5,10
Montana,39.5,57.5,2,10/27/24,38.5,58.4,4
Nebraska,40,55,2,10/29/24,39.1,59.6,2
Nebraska-1,44,50,2,10/29/24,42.5,55.5,1
Nebraska-2,53,43,2,10/29/24,51.3,46.7,1
Nebraska-3,23,71,2,10/29/24,22.4,76,1
Nevada,47.6,48.2,13,11/4/24,47.5,50.6,6
New Hampshire,50.5,45.5,4,11/3/24,50.9,48.1,4
New Jersey,54.7,38.3,3,11/4/24,52,46.1,14
New Mexico,49.8,43.8,4,11/4/24,51.9,45.9,5
New York,57.3,39.7,3,11/4/24,56.4,43.6,28
North Carolina,47.3,48.6,16,11/4/24,47.8,51,16
North Dakota,32,59,1,10/2/24,30.8,67.5,3
Ohio,44.3,52,6,11/4/24,43.9,55.2,17
Oklahoma,40,56,1,9/5/24,31.9,66.2,7
Oregon,53,41,1,10/18/24,55.6,41.3,8
Pennsylvania,48.2,48.2,25,11/4/24,48.7,50.4,19
Rhode Island,54,40,1,11/3/24,55.7,42.1,4
South Carolina,42,53.7,3,10/31/24,40.4,58.2,9
South Dakota,34,60.5,2,10/24/24,34.2,63.4,3
Tennessee,35,56,1,10/15/24,34.5,64.2,11
Texas,44.4,51.8,5,11/3/24,42.4,56.3,40
Utah,32,57.5,2,10/30/24,37.8,59.4,6
Vermont,63,31,1,11/3/24,64.4,32.6,3
Virginia,49.2,43.4,5,11/4/24,51.8,46.6,13
Washington,55.8,36.5,4,11/4/24,57.6,39.3,12
West Virginia,34,61,1,8/30/24,28.1,70,4
Wisconsin,48.8,47.7,17,11/4/24,48.8,49.7,10
Wyoming,27.5,66,2,11/3/24,26.1,72.3,3
```