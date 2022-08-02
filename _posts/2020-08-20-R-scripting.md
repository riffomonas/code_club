---
layout: post
title: "Running R scripts from the command line"
blurb: "You've probably run R code in R, but what about from the command line?"
author: "PD Schloss"
date: 2020-08-20 11:55
comments: false
start_hash: b4b9c4d
youtube: UBopyjT39rY
---

Are you ready for some R? To this point, I've been showing you how you can process data files in bash using reproducible practices. Now, we're going to switch tools and start working with data in R. This will move us closer to quantifying how sensitive and specific amplicon sequence variants are for differentiating bacterial populations in microbiome analyses.

Perhaps you already know a little R. If not, I have several tutorials at the riffomonas.org website that you can use for free to learn a lot of fundamentals for analyzing microbiome and other types of data. Typically, when people analyze data in R, they enter commands at the prompt in RStudio. This is a bit like how we could run bash commands from the command line interface. It works and is very powerful. But it isn't automated and isn't an approach that will play well with a Makefile. In today's episode of Code Club, I'll show you how to create an executable R script that can be run without starting, takes input, and uses good reproducible practices. We'll see how we can integrate this new script with our earlier analysis.

Even if you're only watching this video to learn more about R and don't know what a 16S rRNA gene is, I'm sure you'll get a lot out of today's video. Please take the time to follow along on your own computer. If you haven't been following along but would like to, please check out the notes below where you'll find instructions on catching up, reference notes, and links to supplemental material. You can find [my version of the project](https://github.com/SchlossLab/Schloss_rrnAnalysis_mSphere_2020) on GitHub.



<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



## Important things to remember

* Best practices
  - Include a shebang line as the first line
  - Create a commented header to state the title of the script, author, any inputs and outputs
  - Use comments to describe what is going on in the code
	- Use `.R` as the file extension to make it clear that its a shell script
	- Run `chmod +x filename.R` to make the file executable. This allows you to run the script using `./code/filename.R`

* "shebang" or "bang" line
	- Tells computer where to find bash for running our code
  - `#!/usr/bin/env Rscript --vanilla`. The `--vanilla` tells Rscript to run without saving or restoring anything in the process.

* Inputs
  - You can `args=commandArgs(trailingOnly=TRUE)` to get variables from command line
  - Then use `args[1]`, `args[2]`, etc. to get individual parameters


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
