---
layout: post
title: "Self joining data frames with <code>inner_join</code>"
blurb: "Using self joins to traverse a tree structure"
author: "PD Schloss"
date: 2020-09-28 11:55
comments: false
start_hash: 676d378a1f95689d2332becc9a527be2a601b271
youtube: f6UUS9d9BEE
---

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

You can find [my version of the project](https://github.com/SchlossLab/Schloss_rrnAnalysis_mSphere_2020) on GitHub.


## Important things to remember

* [dplyr package](https://dplyr.tidyverse.org)
* [dplyr cheat sheet (see second page for join information)](https://github.com/rstudio/cheatsheets/raw/master/data-transformation.pdf)


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
