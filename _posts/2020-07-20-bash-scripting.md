---
layout: post
title: "Scripting an analysis from the command line"
author: "PD Schloss"
date: 2020-07-20 14:00
blurb: "Anyone can write a computer program!"
comments: false
start_hash: acd9053
---

Normally when we think about computer programs, we think of something like a web browser or an app on your phone. In the past few episodes we have used a variety of programs including git and atom. But really, a computer program is really a set of instructions telling a computer how to do something. It can be really simple or quite complex. I think of a computational research project as writing a computer program. At first it doesn't seem very complicated. A few commands to get my raw data, a few more to process the raw data, and a few more to generate figures and run statistical tests. By slowly building up the project, my project becomes a pretty complicated computer program. This is not a metaphor! We have the ability to piece together different programs as well as programs we write to instruct the computer how to convert raw data into a finished product. You might think that you can't possibly write your own computer program. You can! In today's episode we will take the first steps of converting our work from the past few episodes into small programs that we can execute from our command line interface.

We may not always appreciate it, but when we're working at the command line, we are using a terminal program but also a program that provides the command line interface called bash. Today we will learn how to write simple programs, also called scripts to automatically download our data and put it in the correct location. As we go forward through our project, we will use this approach to create an automated and reproducible workflow. Writing these types of programs is critical to achieving our overarching goal of understanding the sensitivity and specificity of amplicon sequence variants. Even if you don't have a clue what amplicon sequence variants are, I'm sure you'll get a lot out of this episode. Also, at the end of this episode, I have several exercises for you to work on. If you haven't been following along but would like to, please check out the blog post that accompanies this video where you will find instructions on catching up.

Please take the time to watch today's episode, follow along on your own computer, and attempt the exercises. Don't worry if you aren't sure how to solve the exercises, at the end I will provide solutions.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/NHOG2R-2PZM" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

The reference notes and links that follow are a supplement to the material in the video. The exercises and their solutions are provided at the bottom.

## Important things to remember

* Command line interfaces
  - `bash` and `zsh` are most common
  - For our purposes, the differences aren't important (we'll use `bash`)
  - Both are installed on any system that uses Mac OS X or Linux (including Windows 10's bash system). Starting with Mac OS X Catalina, `zsh` is the default

* Best practices
  - Include a shebang line as the first line
  - Create a commented header to state the title of the script, author, any inputs and outputs
  - Use comments to describe what is going on in the code
	- Use `.sh` as the file extension to make it clear that its a shell script
	- Run `chmod +x filename.sh` to make the file executable. This allows you to run the script using `./code/filename.sh`

* "shebang" or "bang" line
	- Tells computer where to find bash for running our code
  - `#!/usr/bin/env bash` or `#!/bin/bash` - the first is more "portable"

* Inputs
  - You can use `$1`, `$2`, etc to get variables from command line
  - If you want to use those variables in your script, it's best to wrap them in double quotes, not single quotes


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

## Exercises

1\. Create and close an issue to write a script that installs mothur
- You will need to borrow code from what we did in the last episode and adapt it using the instructions we stored in `code/README.md`
- The installation script should be in `code/` and retained under version control, but make sure that `code/mothur/` is not under version control
- Double check that running `code/mothur/mothur '#align.seqs(fasta=data/raw/rrnDB-5.6_16S_rRNA.fasta, reference=data/references/silva_seed/silva.seed_v138.align, flip=T)'` works before closing the issue


2\. We need to align our sequences to make sure they're in the correct orientation and start and end at the same alignment coordinates. Also, if they're aligned, then it will be easier to extract variable regions from all of the sequences. Create and close an issue that aligns sequences our rrnDB fasta sequences to our SILVA SEED reference alignment
- Create `align_sequences.sh` that runs `code/mothur/mothur '#align.seqs(fasta=data/raw/rrnDB-5.6_16S_rRNA.fasta, reference=data/references/silva_seed/silva.seed_v138.align, flip=T)'`
- Run it from the command line, make sure everything works
- add `\*logfile` to `.gitignore`
