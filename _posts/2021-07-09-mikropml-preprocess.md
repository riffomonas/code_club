---
layout: post
title: "Building machine learning models in R with mikropml: preprocessing data (CC125)"
blurb: "Preprocessing data with mikropml"
author: "PD Schloss"
date: 2021-07-09 11:00
start_hash: 21bc222922c8683a776514ff563e2103cb336c1b
end_hash: 8d4d8b100369746d1eb459dd8f0043ab0ee6496b
comments: true
youtube: Xbvqdl_krJQ
---

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
