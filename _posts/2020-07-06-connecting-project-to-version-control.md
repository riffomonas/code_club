---
layout: post
title: "Setting up and using version control for a project"
author: "PD Schloss"
date: 2020-07-06 11:45
blurb: "If we want to track the history of our project, it's best to start from the beginning"
comments: false
---

In the last episode of Code Club we started a new project. It was a very exciting day! The project we're going to be working on over the next Code Club episodes seeks to determine to what degree do inter- and intra-genomic variation limit the interpretation of amplicon sequence variants (ASVs). As I mentioned, this is a newish approach to analyzing 16S rRNA gene sequence data that is all the rage, but not enough people are questioning the assumptions that are baked into the method. We're going to test the assumptions that ASVs represent a biologically coherent entities. We also got familiar with our command line interface using a unix style environment and set up a basic organization structure for our project. Along the way we’ll learn different elements of what it takes to make an analysis reproducible and how to use a variety of tools that will help out. Even if you don’t find the problem we are studying interesting, you will hopefully find the approach generalizable to a variety of problems that do interest you. For today's episode we'll add version control to our project and learn to use git and GitHub to track the history of our project. git is one of those things that can be very confusing, but it doesn't need to be. We'll take it slow and we'll be using git in nearly every episode that follows, so we'll get a lot of practice!

Please take the time to watch today's episode, follow along on your own computer, and attempt the exercises. Don't worry if you aren't sure how to solve the exercises, at the end I will provide solutions. The reference notes and links that follow are a supplement to the material in the video.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/DnwEaa5QtpI" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Version Control

If you're like me, you use the undo function in Microsoft Word or Excel countless times a day. Sometimes, I need to undo a large number of things to get back to a point before I screwed things up. One problem with the undo function is that if I quit the program, the thread of changes is broken and I can't undo something from a previous session in the program. If we use it right, version control is a lot like the undo function. The big difference is that version control won't lose the thread of changes when we quit an application. It will also keep track of my changes across many files in a project. Even better, it works well with text files like those we'll be using for our project. Here's a brief list of things that version control will help us with:

* Documenting the history to a project - When, why, and how did I make a change?
* Back up system - I can delete a file or even my entire project directory and get it back
* Facilitates making your work public - I can use version control to share my code with others
* Enables collaboration -  Other people can make suggestions to improve my project

To achieve these benefits, we'll use the most popular version control tool called `git`. We will be running `git` on our computer to maintain what we call our local repository or "local". Note that even if you doing a project on a computer that's not in the same room as the computer you're typing on, I'll call it your local repository. We'll also make use of GitHub to host our repository as a remote repository or "remote". Here are the `git` commands we'll use for today's episode and somethings you should do to get set up in GitHub.

### git
Watching the video, you'll see me work through a handful of `git` commands. About 95% of what I do with `git` uses 5 commands. So if you can master those, you'll be in good shape. If you're on a Windows computer, be sure you have the [the Ubuntu Linux BASH shell for Windows 10](https://itsfoss.com/install-bash-on-windows/) installed. If you're using Mac OS X, we'll be using the Terminal app so much that you probably want to keep it in your Dock. Here are the commands we'll use.

* `git config` - set up your local git environment (typically copy and paste commands from elsewhere). Be sure to customize values in quotes for you

	```
	git config --global user.name "Your Name"
	git config --global user.email "your_name@email.com"
	git config --global github.user "your_github_account_name"
	git config --global core.autocrlf input	#for mac os x
	git config --global core.autocrlf true	#for windows
	git config --global core.editor "nano -w"
	git config --global --list
	```

* `git init` - create a git repository (look for `.git/` when you use `ls -a`)
* `git status` - what files are not being tracked? which have been changed? which have been staged?
* `git add` - stage files that are not being tracked and stage changes for committing
* `git commit` - commit changes to your repository
* `git pull` - get a copy of your remote repository (i.e. GitHub) and merge it with your local repository
* `git push` - send your local repository to the remote (i.e. GitHub)


### GitHub
GitHub is the most popular website for hosting `git` repositories and we will be using it to host our project's repository. If you don't have an account already, go ahead and set up a [GitHub account](https://github.com). You will get unlimited public repositories for free. If you want to keep your repositories private, you will need to create a paid subscription. However, if you are an educator or student, you can [apply for an academic discount](https://docs.github.com/en/github/teaching-and-learning-with-github-education/applying-for-an-educator-or-researcher-discount). This will give you many benefits including unlimited private repositories. If you would prefer to keep your repository private while you are developing your ideas, then I would hope that once you are done with your project you will be willing to convert your private repository into a public repository.

You can connect to GitHub from the command line interface using https or SSH. In the video we'll use https, which requires us to enter a password. We'll use a setting that only requires us to enter our password once an hour. You can learn how to use your computer's [password manager](https://docs.github.com/en/github/using-git/caching-your-github-credentials-in-git). Alternatively, you can use a SSH key to connect to your repositories. I find this to be the most convenient, but it is a bit more involved to set up. GitHub has some [great instructions](https://docs.github.com/en/github/using-git/caching-your-github-credentials-in-git) to get you going.


## Exercises
1\. As an exercise in the last episode we added `README.md` files to each of our directories. Commit these files to your repository and push the changes to your GitHub account.

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```
git add code/README.md data/README.md data/mothur/README.md data/processed/README.md data/raw/README.md data/references/README.md exploratory/README.md submission/README.md
git commit -m "Create blank README files to preserve organization structure"
git pull
git push
```
</div>


2\. I would like to use a MIT license for this project because we will have a fair amount of code, it is a simple license, and it will allow anyone to use the code and develop it further as long as they provide attribution to us by including the license. You can find the text of the license at https://opensource.org/licenses/MIT. Copy the text of the license into your `LICENSE.md` file and make no changes to the file. Save the text and commit the change to your repository.

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
Copy this into `LICENSE.md` using `nano`

```
Copyright <YEAR> <COPYRIGHT HOLDER>

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
```

Then run the following commands at the prompt

```bash
git add LICENSE.md
git commit -m "Use MIT License for project"
```
</div>


3\. Edit the `LICENSE.md` file to indicate the year and your name as the copyright holder (if we're doing this in parallel, I'm not sure who the true copyright holder is!). Go ahead and commit the change to your repository. Push the changes to your GitHub account.

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
In `nano` edit `LICENSE.md` to change `<YEAR>` to the current year (e.g. 2020) and `<COPYRIGHT HOLDER>` to your name (or mine). Then do the following

```bash
git add LICENSE.md
git commit -m "Update copyright information"
git pull
git push
</div>
