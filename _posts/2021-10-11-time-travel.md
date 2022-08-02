---
layout: post
title: "Easy ways to go back in your git commit history with RStudio (CC153)"
blurb: "Creating a parallel analysis with git"
author: "PD Schloss"
date: 2021-10-11 11:00
comments: false
youtube: Y9h-1u6uQ6c
start_hash: 6dc5b27c08b9e2f3f0050c6cc9a1161dc42c0479
end_hash: 32a63b2b58838805895f5e93a59466ea149705ba
---

Ever wonder how to go back in your git commit history? In this episode of Code Club, Pat shares two approaches of reliving your past code without having to use any hard to remember git syntax. He'll do it all from within RStudio and GitHub. The goal will be to go back to a previous commit, create a branch, and move forward on that branch to create a new type of slope plot. The data depict the percentage of people in 15 countries who would be willing to receive the COVID-19 vaccine as of August and October of 2020.

In this episode, Pat uses RStudio, GitHub, and git.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/vaccination_attitudes/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/vaccination_attitudes/tree/{{ page.end_hash }})


### Data

The `august_october_2020.csv` data is available in the [GitHub repository](https://raw.githubusercontent.com/riffomonas/vaccination_attitudes/3f39b9e09618144874ced760c9a6332498e3a19c/august_october_2020.csv).

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
