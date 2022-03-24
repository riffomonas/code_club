---
layout: post
title: "Creating multiple ROC curves in R with custom functions and map, map_dfr, and map2 (CC123)"
blurb: "Building ROC curves"
author: "PD Schloss"
date: 2021-07-05 11:00
start_hash: 46f2d22d6668933736bf93db6a39f8432da9a99f
comments: true
youtube: X3mRjpt8ZVA
---

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

This is the R script that I generated in this episode

```R
library(tidyverse)
library(broom)
library(ggtext)

set.seed(19760620)

shared <- read_tsv("raw_data/baxter.subsample.shared",
         col_types = cols(Group = col_character(),
                          .default = col_double())) %>%
  rename_all(tolower) %>%
  select(group, starts_with("otu")) %>%
  pivot_longer(-group, names_to="otu", values_to="count")

taxonomy <- read_tsv("raw_data/baxter.cons.taxonomy") %>%
  rename_all(tolower) %>%
  select(otu, taxonomy) %>%
  mutate(otu = tolower(otu),
         taxonomy = str_replace_all(taxonomy, "\\(\\d+\\)", ""),
         taxonomy = str_replace(taxonomy, ";unclassified", "_unclassified"),
         taxonomy = str_replace_all(taxonomy, ";unclassified", ""),
         taxonomy = str_replace_all(taxonomy, ";$", ""),
         taxonomy = str_replace_all(taxonomy, ".*;", "")
         )


metadata <- read_tsv("raw_data/baxter.metadata.tsv",
         col_types=cols(sample = col_character())) %>%
  rename_all(tolower) %>%
  rename(group = sample) %>%
  mutate(srn = dx_bin == "Adv Adenoma" | dx_bin == "Cancer",
         lesion = dx_bin == "Adv Adenoma" | dx_bin == "Cancer" | dx_bin == "Adenoma")

composite <- inner_join(shared, taxonomy, by="otu") %>%
  group_by(group, taxonomy) %>%
  summarize(count = sum(count), .groups="drop") %>%
  group_by(group) %>%
  mutate(rel_abund = count / sum(count)) %>%
  ungroup() %>%
  select(-count) %>%
  inner_join(., metadata, by="group")

sig_genera <- composite %>%
  nest(data = -taxonomy) %>%
  mutate(test = map(.x=data, ~wilcox.test(rel_abund~srn, data=.x) %>% tidy)) %>%
  unnest(test) %>%
  mutate(p.adjust = p.adjust(p.value, method="BH")) %>%
  filter(p.adjust < 0.05) %>%
  select(taxonomy, p.adjust)


composite %>%
  inner_join(sig_genera, by="taxonomy") %>%
  mutate(rel_abund = 100 * (rel_abund + 1/20000),
         taxonomy = str_replace(taxonomy, "(.*)", "*\\1*"),
         taxonomy = str_replace(taxonomy, "\\*(.*)_unclassified\\*",
                                "Unclassified<br>*\\1*"),
         srn = factor(srn, levels = c(T, F))) %>%
  ggplot(aes(x=rel_abund, y=taxonomy, color=srn, fill=srn)) +
  geom_vline(xintercept = 100/10530, size=0.5, color="gray") +
  geom_jitter(position = position_jitterdodge(dodge.width = 0.8,
                                              jitter.width = 0.5),
              shape=21) +
  stat_summary(fun.data = median_hilow, fun.args = list(conf.int=0.5),
               geom="pointrange",
               position = position_dodge(width=0.8),
               color="black", show.legend = FALSE) +
  scale_x_log10() +
  scale_color_manual(NULL,
                     breaks = c(F, T),
                     values = c("gray", "dodgerblue"),
                     labels = c("Healthy", "SRN")) +
  scale_fill_manual(NULL,
                     breaks = c(F, T),
                     values = c("gray", "dodgerblue"),
                     labels = c("Healthy", "SRN")) +
  labs(x= "Relative abundance (%)", y=NULL) +
  theme_classic() +
  theme(
    axis.text.y = element_markdown()
  )

ggsave("figures/significant_genera.tiff", width=6, height=4)

get_sens_spec <- function(threshold, score, actual, direction){

  # threshold <- 100
  # score <- test$score
  # actual <- test$srn
  # direction <- "greaterthan"

  predicted <- if(direction == "greaterthan") {
    score > threshold
    } else {
      score < threshold
    }

  tp <- sum(predicted & actual)
  tn <- sum(!predicted & !actual)
  fp <- sum(predicted & !actual)
  fn <- sum(!predicted & actual)

  specificity <- tn / (tn + fp)
  sensitivity <- tp / (tp + fn)

  tibble("specificity" = specificity, "sensitivity" = sensitivity)
}

get_roc_data <- function(x, direction){

  # x <- test
  # direction <- "greaterthan"

  thresholds <- unique(x$score) %>% sort()

  map_dfr(.x=thresholds, ~get_sens_spec(.x, x$score, x$srn, direction)) %>%
    rbind(c(specificity = 0, sensitivity = 1))

}

# get_sens_spec(100, test$score, test$srn, "greaterthan")
# get_roc_data(test, "greaterthan")

roc_data <- composite %>%
  inner_join(sig_genera, by="taxonomy") %>%
  select(group, taxonomy, rel_abund, fit_result, srn) %>%
  pivot_wider(names_from=taxonomy, values_from=rel_abund) %>%
  pivot_longer(cols=-c(group, srn), names_to="metric", values_to="score") %>%
  # filter(metric == "fit_result") %>%
  nest(data = -metric) %>%
  mutate(direction = if_else(metric == "Lachnospiraceae_unclassified",
                             "lessthan","greaterthan")) %>%
  mutate(roc_data = map2(.x = data, .y=direction, ~get_roc_data(.x, .y))) %>%
  unnest(roc_data) %>%
  select(metric, specificity, sensitivity)

roc_data %>%
  ggplot(aes(x=1-specificity, y=sensitivity, color=metric)) +
  geom_line() +
  geom_abline(slope = 1, intercept = 0, color="gray") +
  theme_classic()

ggsave("figures/roc_figure.tiff", width=6, height=4)
```

## Installations

If you haven't been following along, you can get caught up by doing the following:

* [(windows) Install the Ubuntu Linux BASH shell for Windows 10](https://itsfoss.com/install-bash-on-windows/)
* (mac) Install `homebrew` and `git`
  ```
	/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
	brew install git
	```

* To get to where we are at the beginning of this episode (you won't have the same issue numbers at Pat)...
  - Set up a [GitHub](https://www.github.com) account
  - Create a new [GitHub repository](https://github.com/new)
    - Call it "mikropml_demo"
    - Make it Public
    - Don't check the box next to "Initialize this repository with a README"
    - Click the green "Create repository" button
  - Go to your command line and enter the following replacing `<your_github_id>` with your GitHub user id

		git clone git@github.com:SchlossLab/mikropml_demo.git
		cd mikropml_demo
		git reset --hard {{ page.start_hash }}
		git remote set-url origin git@github.com:<your_github_id>/mikropml_demo.git
		git push -u origin master

  - Return to GitHub and refresh your browser.
  - Go to the `mikropml_demo` directory on your computer and double click on the `mikropml_demo.Rproj` icon. This will launch RStudio and you'll be good to go.
