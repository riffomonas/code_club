---
layout: post
title: "Using GitHub actions to run Snakemake pipelines (CC260)"
blurb: "Reproducible pipelines with GitHub actions"
author: "PD Schloss"
date: 2022-10-27 11:00
comments: false
youtube: t1MGEVeTgQM
start_hash: 743580377c34ab883bc17a83e5ea9162d55c09c7
end_hash: 2b09190f6f64d72094fa637a69f5b55a15470c87
---

Pat shows how we can run a snakemake workflow with conda/mamba environments with GitHub actions to rerun a data analysis pipeline and then commit the changes to our repository so that an updated webpage is generated each day. This episode ties together many principles in reproducibility including conda and mamba environmens, scripting, R, snakemake, version control, and automation. The overall goal of this project is to highlight reproducible research practices using a number of tools. The specific output from this project will be a map-based visual that shows the level of drought across the globe. The website we generate in this episode can be found at https://www.riffomonas.org/drought_index.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

You can browse the state of the repository at the

* [beginning of the episode](https://github.com/riffomonas/drought_index/tree/{{page.start_hash}})
* [end of the episode](https://github.com/riffomonas/drought_index/tree/{{page.end_hash}})
