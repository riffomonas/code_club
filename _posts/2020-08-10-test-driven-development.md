---
layout: post
title: "The value of Test Driven Development in data analysis"
blurb: "Measure twice, cut once"
author: "PD Schloss"
date: 2020-08-10 11:55
comments: false
start_hash: 863a98a3c
youtube: h_qFha7vu9I
---

In last Monday's episode of Code Club I did something that I do regularly. I screwed up. [Rasmus Kirkegaard](https://www.youtube.com/watch?v=yORbSp7Fe10&lc=UgyLCbeiVD4D2AoStBJ4AaABAg) noticed the problem and was kind enough to point out the problem in the comments to that episode. Part of my motivation for making these videos and my general approach to teaching is to normalize mistakes. There are a lot of YouTube tutorials on how to use grep, sed, and every other bash command. The problem is that they're presented in a way that is too abstract. Those tutorials don't show how those commands interact with the output from other commands. They're highly edited and don't show typos, goofs, or how to identify and solve problems.

Not mine! If you've watched along, you've seen numerous typos and redos. These are not part of the act. They're the reality of how I and every other person analyzes data. What I have found is that experience brings the ability to diagnose and solve problems. This process is also missing from from most tutorials. In today's episode of Code Club, I'm going to show you how I investigated the problem Rasmus pointed out and resolved it. I'm sure I would have found the problem eventually, but by doing these tutorials publicly, we were able to figure it out much faster. Thanks, Rasmus!

As we are doing steps in a data analysis we get a little cocky. We think things are going to work the way we expect. You may recall in last Monday's episode that there were a few sequences that started and ended with periods to indicate missing data. I wrote a `sed` command to convert those periods to hyphens to represent gaps in the alignment. As we were going through my solutions to the exercises, I found a bug in that sed command and thought I fixed it. I even checked the output. Unfortunately, I was flustered and rushing. Instead of checking the output for a region that had those leading periods, I checked the output of the full length sequences, which did not have the problem.

Today I'm going to present the approach that I should have taken. It's related to a concept that is commonly used in programming called Test Driven Development (TDD). It isn't as widely used in data analysis, but there are ideas in Test Driven Development that we can draw from to make our analysis more robust. The idea is that we start with a set of tests that fail. We then write code to generate output that passes the test. If we later find a situation that produces the wrong output, then we add that situation to our set of tests and modify the code so the test passes. Because modifying the code can cause other tests to fail, every time the code is updated the tests are rerun. This sounds a bit like make, right? Programming languages including R and Python have frameworks that make Test Driven Development much easier to execute. I'm not aware of such a framework for bash. In today's episode we're going to figure out where the problem is, create a set of test sequences that trigger the problem, and then modify our code to resolve the problem. Along the way we'll learn more about sed and grep.

Even if you're only watching this video to learn more about bash commands and don't know what a 16S rRNA gene is, I'm sure you'll get a lot out of today's video. Please take the time to follow along on your own computer and attempt the exercises. Don't worry if you aren't sure how to solve the exercises, at the end of the video I will provide solutions. If you haven't been following along but would like to, please check out the notes below where you'll find instructions on catching up, reference notes, and links to supplemental material. You can find [my version of the project](https://github.com/SchlossLab/Schloss_rrnAnalysis_mSphere_2020) on GitHub.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Important things to remember

### `grep`
* `-B #`: return the # lines that occur before the line that matches the pattern
* `-A #`: return the # lines that occur after the line that matches the pattern
* `[.\A]`: match the characters in the brackets
* `[^.\A]`: match anything that does not contain the characters in the brackets

### `sed`
* Excellent [sed tutorial](https://www.grymoire.com/Unix/Sed.html)
* `sed '/pattern_a/ s/pattern_b/replacement/'` - on lines containing `pattern_a`, replace `pattern_b` with `replacement`
* `sed '/pattern_a/ !s/pattern_b/replacement/'` - on lines *not* containing `pattern_a`, replace `pattern_b` with `replacement`



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

  - Return to GitHub and refresh your browser
  - Within `Schloss_rrnAnalysis_XXXX_2020/` run

	```
	make data/v19/rrnDB.align
	make data/v4/rrnDB.align
	```

  - Then you should now be good to go


## Exercises

1\. Write a pipeline to generate a fasta file that contains the 21 copies of the V4 region of the 16S rRNA gene from *Photobacterium damselae* (GCF_003130755.1)

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```
grep -A 1 "GCF_003130755.1" data/v4/rrnDB.align | grep -v "^--$" > photobacterium_damselae.fasta
```
</div>

2\. Write a `sed` statement to unalign the sequences in `data/v4/rrnDB.align`. How would you test that it works?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```
sed "/^[^>]/ s/[.-]//g" test.fasta
sed "/^[^>]/ s/[.-]//g" data/v4/rrnDB.align
```
</div>

3\. Add a test statement to `code/extract_region.sh` to make sure that none of the sequences have periods in them.
