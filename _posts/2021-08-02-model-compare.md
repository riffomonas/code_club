---
layout: post
title: "Using ggplot2 to group x-axis discrete values into subgroups (CC133)"
blurb: "Creating multi grouped x-axis titles and labels"
author: "PD Schloss"
date: 2021-08-02 11:00
start_hash: 617a21c15924de143c3d5c71ef90d3b25aa92f80
end_hash: f5d8afe98b6d0586cf51aff6c7792e270e58fad4
comments: true
youtube: KoAJZYEOwSE
---

When we group sets of variables into subgroups on the x-axis we signal to our audience that they need to compare the values within the grouping. But how do we do this with ggplot2? In this episode of Code Club, Pat will show how you can great four (!) layers of subgroups by using the ggplot2 facet_wrap function. At the end, we'll revisit our code and see how we can make it more DRY.

Pat will use functions from the `mikropml` R package and the `facet_wrap` function from `ggplot2` as well as functions from `dplyr` and `purrr` in `RStudio`.


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
