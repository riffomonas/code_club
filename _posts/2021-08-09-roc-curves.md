---
layout: post
title: "How to pool ROC curves in R to better understand a model's performance (CC135)"
blurb: "Comparing machine learning models"
author: "PD Schloss"
date: 2021-08-09 11:00
start_hash: 473f3da8502b49f1382e16826e6bcfc7b361b930
end_hash: 627f2131a1afe9596a6cfd8ccea5c44c5c4c52c3
comments: true
youtube: UFvL2NXLvD8
---

In this Code Club, Pat shows how he would pool ROC curves so that you can directly assess a model's sensitivity for specificity. The area under the receiver operator characteristic (ROC) curve (AUC) is a useful metric of performance, but it isn't always the best way to assess performance since it looks over all possible specificities. The challenge is that with the mikropml framework we get one ROC curve per 80/20 training-testing split and we need to pool the curves to get a composite ROC curve. Even if you don't care about ROC curves, this episode is sure to have a lot of value for you including a little known R tip towards the end of the episode!

Pat will use functions from the `mikropml` R package and the `ggplot2` and `dplyr` packages in `RStudio`.


<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

You can browse the state of the repository at the
* [beginning of the episode](https://github.com/riffomonas/mikropml_demo/tree/{{ page.start_hash }})
* [end of the episode](https://github.com/riffomonas/mikropml_demo/tree/{{ page.end_hash }})


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
    - Call it "Schloss_rrnAnalysis_XXXX_2020" (feel free to use your own last name)
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
