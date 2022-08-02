---
layout: post
title: "Measuring the sensitivity and specificity of ASVs for genomes"
blurb: "Grouping and summarizing our data with <code>dplyr</code>"
author: "PD Schloss"
date: 2020-09-10 11:55
comments: false
start_hash: 1897d55f38fd05e4c62e4ec8b780bf8e42308876
youtube: 7-Ju_5Dk6Uk
---

A super common task in analyzing data is to measure summary statistics for different groups of items. We might want to know the average age for Republicans versus Democrats. Perhaps we want to know whether men have higher salaries than women. Maybe we want to determine how salaries vary by level of education. Each of these questions requires us to take a set of data, subset the data by a group, and then calculate a value for each group. How do we do this?

One approach might be to extract the data corresponding to Republicans from the dataset and then calculate the average age. We could then repeat for the Democrats. Let's say there was a third party we were interested in, we'd repeat the process. But, what if there's a fourth party? Fifth? This would get tedious really quick. There's an alternative approach available to us using the `dplyr` package, which is part of the `tidyverse`.

The `dplyr` approach is super powerful. We can use the `group_by` function to group our data by a categorical variable like political party. We can then use the `summarize` function to calculate summary statistics for each value of the categorical variable. This approach is super adaptable to any changes in the data. I don't need to know all of the political parties to carry out this process, which is part of what makes it so powerful. Furthermore, we can apply it to a wide array of questions.

If you've forgotten what we've been trying to do over the past 17 episodes, I don't blame you! A lot of the work in a data analysis pipeline is getting data into a format that allows us to answer our questsions. You might recall, bacterial genomes tend to have more than one copy of the 16S rRNA gene. This raises a few questions. First, how many copies of the gene do bacterial genome typically have? Second, if there are multiple copies of the gene in a genome, are they identical to each other or are there differences? Finally, if we find a sequence in one genome, how often might we find it in another genome? These questions are important to the ongoing discussion in microbial ecology because there is a push to using distinct sequences or amplicon sequence variants (ASVs) as surrogates for bacterial taxa rather than using clusters of related sequences as surrogates for bacterial taxa.

Let's think about the data as we currently have it. We have a column that contains an identifier for each 16S rRNA gene sequence, that is the ASV, that was found in the rrnDB database. We have a second column that contains an identifier for each genome in the database. Finally, we have a column that indicates the number of times each ASV was found in each genome. But how does this relate back to the examples I started with? How is this similar to grouping people by political party? If I group the data by genome, I could count the number of 16S rRNA genes in each genome. I could also count the number of ASVs in each genome. Alternatively, if I group the data by ASV, I could count the number of genomes where each ASV was found.

We're really going to leverage this approach in future episodes. Today, I've been talking about using full length sequences, but I could also use a sub-region of the gene and see how these counts change. Instead of looking at the genome level, I could also look at the species, genus, or any higher taxonomic level as well. But we'll look at how these counts vary by taxonomic level for a future episode. For today's episode we'll use the `group_by` and `summarize` functions and a few other helper functions to do an exploratory analysis of the sensitivity and specificity of ASVs to each genome.

Even if you're only watching this video to learn more about R and don't know what a 16S rRNA gene is, I'm sure you'll get a lot out of today's video. Please take the time to follow along on your own computer. If you haven't been following along but would like to, please check out the notes below where you'll find instructions on catching up, reference notes, and links to supplemental material. You can find [my version of the project](https://github.com/SchlossLab/Schloss_rrnAnalysis_mSphere_2020) on GitHub.



<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



## Important things to remember

* General functions
  - `group_by`: group a data frame by one or more columns
  - `summarize`: summarize data within the level of each grouping variable
  - `count`: count the number of times each level of a categorical variable are used
  - `mutate`: create a column
* Helper functions
  - `median`: calculate the median
  - `mean`: calculate the mean
  - `sum`: add the numbers in a column together
  - `n`: the number of rows in the grouped data
  - `quantile`: find the value for a `prob`ability value


## Installations

If you haven't been following along, you can get caught up by doing the following:

* [(windows) Install the Ubuntu Linux BASH shell for Windows 10](https://itsfoss.com/install-bash-on-windows/)
* (mac) Install `homebrew` and `wget`
  ```
	/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
	brew install wget
	```

* To get to where we are at the beginning of this episode (you won't have the same issue numbers at Pat)...
  - Set up a [GitHub](https://www.github.com) account
  - Create a new [GitHub repository](https://github.com/new)
    - Call it "Schloss_rrnAnalysis_XXXX_2020" (feel free to use your own last name)
    - Make it Public
    - Don't check the box next to "Initialize this repository with a README"
    - Click the green "Create repository" button
  - Go to your command line and enter the following replacing `<your_github_id>` with your GitHub user id

		git clone git@github.com:SchlossLab/Schloss_rrnAnalysis_mSphere_2020.git
		cd Schloss_rrnAnalysis_XXXX_2020
		git reset --hard {{ page.start_hash }}
		git remote set-url origin git@github.com:<your_github_id>/Schloss_rrnAnalysis_XXXX_2020.git
		git push -u origin master

  - Return to GitHub and refresh your browser. Then you'll be good to go.
