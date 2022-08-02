---
layout: post
title: "Understanding model interpretability in R with ggplot2 and mikropml (CC134)"
blurb: "Visualizing the features of machine learning models"
author: "PD Schloss"
date: 2021-08-05 11:00
start_hash: f5d8afe98b6d0586cf51aff6c7792e270e58fad4
end_hash: 473f3da8502b49f1382e16826e6bcfc7b361b930
comments: false
youtube: mzHfjdkG4CQ
---

The interpretability of a machine learning model tends to vary by the performance of the model. The need to interpret your model depends on what you hope to do with that model. In this Code Club, Pat shows how you can extract interpretability data from models created using mikropml and visualize the importance of features that are used in the model.

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
