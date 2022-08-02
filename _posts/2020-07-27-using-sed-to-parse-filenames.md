---
layout: post
title: "You sed what? Modifying variables with sed in bash"
author: "PD Schloss"
date: 2020-07-27 11:55
blurb: "We clean up the code in our Makefile shell scripts"
comments: false
start_hash: 7efa956
youtube: D4ojrF8AeeQ
---

In the last two episodes we've created a few bash scripts and a Makefile to help automate running those scripts while keeping track of the dependencies. You may recall that we aren't tracking changes to our data files because they're too big. But with our bash scripts and make, we can regenerate them if they somehow got deleted or we needed start over. Of course, you should be able to use my files to reproduce what I've already done, which is a win for reproducibility. Because these tools are so powerful, I want to spend a little more time with them.

In today's episode we're going to clean up our scripts and Makefile a bit. If you look at our Makefile and bash scripts for downloading the rrnDB files, you'll see that we pass in the name of the file we want to download. We didn't include the path indicating where it should go once it is extracted. Instead, we hardcoded that it should be outputted to our `data/raw/` directory. If I ended up wanting to change where to put the files, then I'd have to change the bash script rather than the Makefile rule.

Today, we'll learn a bash command, `sed`, that will allow us to give the script the file's name with its path. I admit that this is a relatively small improvement for our overall project. But, along the way we'll learn a few other bash tools including the pipe and how to capture output to a variable. We'll also see how we can use variables that are built into make that simplify our Makefiles. Because these improvements will make our analyses easier to develop, they will also be more reproducible.

Even if you haven't been following along with the past episodes, I'm sure you'll get a lot out of today's video. Please take the time to follow along on your own computer and attempt the exercises. Don't worry if you aren't sure how to solve the exercises, at the end of the video I will provide solutions. If you haven't been following along but would like to, please check out the notes below where you'll find instructions on catching up, reference notes, and links to supplemental material. You can find [my version of the project](https://github.com/SchlossLab/Schloss_rrnAnalysis_mSphere_2020) on GitHub.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Important things to remember

* `sed` - [Excellent resource](https://www.grymoire.com/Unix/Sed.html) that all of my google searches generally bring me to
* Make's [automatic variables](https://www.gnu.org/software/make/manual/html_node/Automatic-Variables.html) - there are many, but these are the only ones I use regularly
  - `$@` - the  name of the target
  - `$^` - the name of all of the prerequisites
  - `$<` - the name of the first prerequisite


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

1\. Can you explain what happens if we leave off the `echo` command in our script? You might remember we got a similar error in the episode where we were developing our bash scripts.

```
path=`$target | sed -E "s/(.*\/).*/\1/"`
```

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
We get the following error message:
```
-bash: data/raw/rrnDB-5.6.tsv: Permission denied
```

This is the same error we got when trying to run a bash script as an executable before we used `chmod +x`. You might have thought this should have run our `sed` command on the contents of the file. To do that, we need a slightly different syntax

```
path=`sed -E "s/(.*\/).*/\1/" < $target`
```
</div>

2\. Write the command to extract the variable region (i.e. V4) from `data/V4/rrndb.align` and save it to a variable named `region`. Try to make your regular expression as general as you can

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```
file=data/V4/rrndb.align
region=`echo $file | sed -E "s/.*\/(.*)\/.*/\1/"`
echo $region
```

or

```
file=data/V4/rrndb.align
region=`echo $file | sed -E "s_.*/(.*)/.*_\1_"`
echo $region
```
</div>

3\. We discussed using the automatic variable `$@` to represent the target name in the recipes of our Makefile. You can also use `$<` to represent the name of the first prerequisite and `$^` to represent all of the names of the prerequisites. How would you edit the rule to generate `data/raw/rrnDB-5.6_16S_rRNA.fasta` and `data/raw/rrnDB-5.6_16S_rRNA.align` so that the recipe only includes these automatic variables? Any thoughts about the revised syntax?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```
data/raw/rrnDB-5.6_16S_rRNA.fasta : code/get_rrndb_files.sh
	$^ $@

data/raw/rrnDB-5.6_16S_rRNA.align : code/align_sequences.sh\
			data/references/silva_seed/silva.seed_v138.align\
			data/raw/rrnDB-5.6_16S_rRNA.fasta									
	$^
```

Although the extra typing is reduced in these revised scripts, the recipe line might be a bit too cryptic. There is a tradeoff between simplicity and ease of reading. Regardless, this exercise highlights that it would be good to have a practice of putting the script prerequisite first followed by the data.
</div>
