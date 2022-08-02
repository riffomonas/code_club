---
layout: post
title: "Laying out a plan for our project"
author: "PD Schloss"
date: 2020-07-09 11:55
blurb: "Plans are worthless, but planning is everything"
comments: false
---

Former US president and Army general, Dwight Eisenhower once said, "Plans are worthless, but planning is everything". That is especially true for research projects. If we don't have a plan for getting somewhere, then we'll be sure to get nowhere. I hate working in the midst of chaos, can you tell? We've spent two episodes getting our project organized and we haven't touched any data or code yet. If you have ever taken over a project from a former member of your research group, then you'll know the frustration of trying to find things and make sense of what they did. Your first step would be to get the project organized. Right?! Well, today we'll also talk a bit more about organization and how we can use issues in GitHub to give ourselves a todo list and use branches in git to systematically work through those issues. Don't worry, today we'll download the data that we'll be using to determine to what degree inter- and intra-genomic variation limit the ability to interpret amplicon sequence variants (ASVs) as a biologically coherent entity that proponents of ASVs claim. As we go along, we'll see the command line and `git` commands that we saw in the last two episodes so that we can continue to practice those skills while learning new `git` tools.

Please take the time to watch today's episode, follow along on your own computer, and attempt the exercises. Don't worry if you aren't sure how to solve the exercises, at the end I will provide solutions. The reference notes and links that follow are a supplement to the material in the video.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/O4he8SUdP1U" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## `git` and GitHub revisited
The notes below are meant to supplement the video presentation of the Code Club episode.


### GitHub flow
[GitHub flow](https://guides.github.com/introduction/flow/) is a process used by software developers that allows them to work on multiple new features at the same time. We can adapt the idea behind GitFlow for data analysis. This will allow us to systematically identify and addressing different steps in our data analysis plan. It makes use of an "issue tracker", which accompanies each GitHub repository. To implement this process, we'll learn a few more git commands including `git branch`, `git checkout`, and `git merge`. We'll learn GitFlow through the process of downloading our reference files.

1. Create an issue in your repository's issue tracker on GitHub
2. Create and check out a branch in your local repository
	```bash
	git branch issue_1
	git branch
	git checkout issue_1
	git branch
	git status
	```
3. Work on the issue. As you go through the issue, feel free to add to the thread for the issue on GitHub
4. Commit the change when you are done with the issue. In the commit message, include the statement "closes #[issue_number]"
5. Checkout your master branch
	```bash
	git checkout master
	```
6. Merge your issue branch into the master branch. If you have conflicts, open the offending files and resolve the conflict and commit the change.
	```bash
	git merge issue_1
	```
7. Push to your remote repository
	```bash
	git push
	```
8. Refresh the issue and see that it has closed or close it yourself



### `.gitignore`

The `.gitignore` file sits in the root of our project directory. This file is a listing of files and directories that we want `git` to ignore. We generally want to ignore things that live inside of `data/` since these tend to be large files. GitHub limits us to a limit of 100 MB per file and 1 GB per repository. Software developers will also recommend ignoring anything that is not code since it can easily be recreated. For bioinformatics data analyses, generating these files can take a long time and significant resources. I tend to commit smaller summary files that others might want to use without the headache of rerunning my analysis themselves.


## Exercises

1\. Create a new issue. In this issue, we want to start accumulating a bibliography of papers describing oligotypes and amplicon and exact sequence variants. As we find new references, we can add them to the issue's thread.

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
* Huse SLP: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2909393/
* Eren Oligotyping: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3864673/
* Callahan: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5702726/
* Fierer heterogeneity: http://fiererlab.org/2017/10/09/intragenomic-heterogeneity-and-its-implications-for-esvs/
* Fierer lumping/splitting: http://fiererlab.org/2017/05/02/lumping-versus-splitting-is-it-time-for-microbial-ecologists-to-abandon-otus/
* Sun heterogeneity: https://aem.asm.org/content/79/19/5962.abstract
* dada2
* unoise
* deblur
* pre.cluster
</div>


2\. Use GitHub flow to resolve Issue 2.

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
* Do the following:
	```BASH
	git branch issue_2
	git checkout issue_2
	git branch
	```
* Go to https://mothur.org/wiki/silva_reference_files/ and download Release 138's "recreated SEED database"
* Decompress file and move it to `data/references/` directory
* Put previous two bullet points into `data/references/README.md` file
* Add `data/references/` to `.gitignore`
* Do the following
	```bash
	git status
	git add
	git status
	git commit
	git checkout master
	git merge issue_2
	git push
	```
* Refresh issue
</div>


3\. Use GitHub flow to resolve Issue 3.

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
* Do the following:
	```BASH
	git branch issue_3
	git checkout issue_3
	git branch
	```
* Go to https://github.com/mothur/mothur/releases/latest and download `Mothur.OSX-10.13.zip`
* Decompress file and move it to `code/` directory
* Put previous two bullet points into `code/README.md` file
* Add a line to `README.md` that `mothur v.1.44.1` is a dependency
* Add `code/mothur` to `.gitignore`
* Do the following
	```bash
	git status
	git add
	git status
	git commit
	git checkout master
	git merge issue_3
	git push
	```
* Refresh issue
</div>
