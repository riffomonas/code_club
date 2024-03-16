---
layout: post
title: "Picking a name for my R package (CC267)"
blurb: "Creating the skeleton for a new R package"
author: "PD Schloss"
date: 2024-03-21 11:00
comments: false
youtube: N1yn2716Hng
start_hash: b90b8be7fb7efb1dd64a318c217983529c1cf5e3
end_hash: 2b630aa71177721611419c978d3da422d2f0fb41
---

Picking a name for an R package is a bit like picking a name for your kid. What I screw up? We have time to finalize the name, so let me know if you don't like what I come up with. This video gives some explanation for the name that I have settled on for now. After picking a name, I create a skeleton for the package including getting it up on GitHub, creating a README, setting up tests, and roughing in documentation. Thankfully, Hadley Wickham and Jenny Bryan have removed a lot of the pain of pulling off these steps with their book R Packages. I'll apply the first chapter of their book to start the development of my R package. We'll use a number of functions from the devtools, usethis, and testthat, packages to develop the skeleton.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

You can browse the state of the repository at the [end of the episode](https://github.com/riffomonas/phylotyper/tree/{{page.end_hash}})

<!-- * [beginning of the episode](https://github.com/riffomonas/drought_index/tree/{{page.start_hash}}) -->
<!-- * [end of the episode](https://github.com/riffomonas/drought_index/tree/{{page.end_hash}}) -->
