---
layout: post
title: "Automating the steps of our analysis plan with make"
author: "PD Schloss"
date: 2020-07-23 11:55
blurb: "Automation tools can provide a reproducibility map for our analysis"
comments: false
start_hash: da1880d
youtube: 8j8pqDKsueI
---

In the last episode, we converted the notes we had been keeping in our README files to bash scripts. This was a great step forward towards having an automated and reproducible analysis. You'll recall that we created scripts for downloading our reference files and files from the rrnDB. In the exercises, we wrote scripts to download and install mothur and to use mothur to align the sequences from the rrnDB against the reference files. This was great because we could call the scripts from  from the command line and everything would be done for us. From my experience, the challenge then becomes remembering the files and scripts that downstream scripts depend on. For example, I anticipate that the rrnDB might update its database this fall. What do I need to run if I want to update the rrnDB files in my database? Our sequence alignment depends on the sequences from the rrnDB and the script that downloads the rrnDB files. So, I would have to run the download script before running the script that aligns the sequences. What would I need to run if SILVA or mothur puts out a new release, which they likely will before the project is done? That's a lot to keep track of and our project has only just begun! Therefore, we need a tool that will keep track of all of these dependencies.

A few years ago, I was at a conference on reproducible research methods and I heard Karl Broman, a statistician at the University of Wisconsin, say that "make" is central to insuring the reproducibility of his work. That got my attention! This was especially true since I had only ever used make to install software from its source code. The more I thought about it, the more using make made sense. make or more officially, GNU Make, is a program that keeps track of dependencies. The developer writes **rules** that tell make what file needs to be created. That file is called a **target**. The rule also indicates the dependencies or **prerequisites** to building the target as well as the **recipe** for generating the target. Thinking back to the example from the last episode, the target is the alignment file generated from mothur, the prerequisites included the rrnDB fasta file, the SILVA SEED reference alignment, and the script that ran the alignment, and the alignment involved running that script. If the target file doesn't exist or any of those dependencies are newer than the target file, then make will re-run the dependency. But what if the script to download the rrnDB file changes? If I tell make to generate my alignment, it will see that the download script is newer than the rrnDB fasta file and will re-download the rrnDB fasta file. This will trigger re-running the rule to generate the sequence alignment.

GNU Make is installed with bash so it's available for people using Mac OS X, Linux, and Windows 10's bash system. I know GNU Make, so I'll be using that in our analysis. An newer alternative is Snakemake, which also seems pretty powerful. But when I was trying learn it, I got frustrated installing it and gave up. Perhaps down the road I'll revisit Snakemake and we can do a head to head comparison with make. For now, I'm pretty happy with make.

As I go forward with this project using Make will become baked into our workflow much like using GitHub flow. As I mentioned earlier, when we install software from source code, we typically enter "make install". My goal for this project isn't to install software. Rather, I want to generate a manuscript that I can submit to a journal. At the end of this project, I hope to be able to write "make manuscript.pdf" and it does everything from downloading my data all the way through generating a PDF version of my manuscript and its figures. If we can do that, then we will have great confidence that our analysis is reproducible. But it would also be great to rerun this analysis with a new version of the rrnDB, SILVA, or mothur. This would allow me to see how robust my analysis is to updates in the database. Not only is reproducibility great for transparency and getting things "right", but it's also great for improving the rigor of our work. Given the growing popularity of amplicon sequence variants in microbial ecology, if I'm going to critique their use, then I want to make sure that my analysis is rigorous and that anyone can reproduce what I am doing. First, I need to show you how to use make! In today's episode we'll ease into the use of make and in future episodes we'll see how we can improve the sophistication of our use of this great tool.

Even if you don't have a clue what amplicon sequence variants are, I'm sure you'll get a lot out of this episode. Please take the time to watch today's episode, follow along on your own computer, and attempt the exercises. Don't worry if you aren't sure how to solve the exercises, at the end of the video I will provide solutions. If you haven't been following along but would like to, please check out the notes below where you'll find instructions on catching up, reference notes, and links to supplemental material. You can find [my version of the project](https://github.com/SchlossLab/Schloss_rrnAnalysis_mSphere_2020) on GitHub.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Important things to remember

* GNU Make reference material
* Makefile should be in your project root directory. You can run `make <target>` from the project's root directory to generate `<target>`.
* Use tabs to indent, not lines!
* Can use `ls -lth` to get more full output from `ls` including when a file was created
* If GNU Make needs to generate a dependency to complete a recipe, it will delete the dependency unless you put `.SECONDARY` somewhere in the Makefile
* GNU Make output
	* `make -n <target>` to see what will run
	* `make: '<target>' is up to date.`
	  - good sign!
	* `make: *** No rule to make target ''<target>'.  Stop.`
	  - Most likely have a typo in your target's name (either in make call or in the recipe's target)
	  - Perhaps you have a dependency that you haven't created a rule for yet or it doesn't exist in your file system (look for typos if you think it should)
	* `Makefile:2: *** missing separator.  Stop.`
	  - Probably using spaces instead of tabs to indent lines


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

1\. Running the following generates an error. What is the error message? How do we resolve the problem?

```
make data/references/silva.seed_v138.align
```

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
This generates the error:

```
```

The problem is the path to the file. The following works:

```
make data/references/silva_seed/silva.seed_v138.align
```
</div>

2\. Our alignment script only has one line. What are the advantages of putting that line into a bash script file versus the body of the recipe?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
Advantages of having a script
* If we change the one liner in the Makefile, it doesn't trigger make to update the target.
* If it's a script, we can add more code to the script, change settings, etc. and because it will become a dependency, these types of changes will trigger make to call the script.

Disadvantages of having a script
* Perhaps a little silly to have a script with only one line in it
</div>


3\. Complete Issue 10
