---
layout: post
title: "Using cut, sort, and unique to explore data with bash"
blurb: "Simple bash commands can help us to do complicated things"
author: "PD Schloss"
date: 2020-08-06 11:55
comments: false
start_hash: 77e314f5
youtube: gPX7d3BcZpg
---

A lot of bioinformatics is getting data from one format to another. Sometimes that conversion can be pretty sophisticated and other times it can be pretty simple. For example, I have a file with a bunch of aligned sequences and I want to know how many times each sequence occurs in each genome. I know that the header for each sequence in my FASTA file has the information I need to know which genome the sequence comes from, but I don't know which field it is. To figure this out, I'm going to use some of my favorite tools to help me. These tools will include `grep`, which we saw in the last episode, as well as `cut`, `sort`, and `uniq`.

Much of what we'll do today you could also do in R or Python. However, using these bash commands will allow me to get to my answer in a single line of code, whereas R or Python will require a lot more effort. The `cut` command "cuts" text out of a line based on how we define different regions of the line. For example, we've seen that the header lines start with the genus and species name of the organism that was sequenced. We previously put an underscore (i.e. `_`) between the genus and species names. We can use `cut` to extract the genus name by pulling out anything that occurs before the underscore. The `sort` command sorts the lines we give the command. We can also sort lines based on specific fields within the line. For example, we could sort header lines based on the GenBank RefSeq accession number. Finally, the `uniq` function deconvolutes a set of lines to return the unique lines. We can also ask it to count the number of times each line occurred in the original set of lines. Hopefully, you can see from my description of these three commands that we could count the number of genera represented in the dataset, the number of 16S rRNA gene copies per genome, or numerous other questions about our data.

We'll use these commands to help us figure out which bit of information between the pipe characters corresponds to a unique identifier for each genome. This will allow us to use mothur's `count.seqs` function to count the number of times each unique 16S rRNA gene sequence appears in each genome and the number of times it appears in more distantly related genomes. If we can generate this information, we can use it to quantify how specific an amplicon sequence variant or ASV is to a genome, species, genus, or any other taxonomic level.

Even if you're only watching this video to learn more about bash commands and don't know what a 16S rRNA gene is, I'm sure you'll get a lot out of today's video. Please take the time to follow along on your own computer and attempt the exercises. Don't worry if you aren't sure how to solve the exercises, at the end of the video I will provide solutions. If you haven't been following along but would like to, please check out the notes below where you'll find instructions on catching up, reference notes, and links to supplemental material. You can find [my version of the project](https://github.com/SchlossLab/Schloss_rrnAnalysis_mSphere_2020) on GitHub.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Important things to remember

### Common `cut` arguments
* `-f`: which field you want
* `-d`: the delimiter used to define fields


### Common `sort` arguments
* `-n`: numerical sort
* `-r`: reverse sort
* `-k`: which field to sort on
* `-t`: the delimiter to define fields based on (default is a space character)


### Common `uniq` arguments
* default: return the unique lines
* `-c`: the number of times each unique line appears


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

1\. How many genomes are in the rrnDB archive that are from the *Thermus*?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
16

```
grep ">Thermus" data/raw/rrnDB-5.6_16S_rRNA.fasta | cut -f 2 -d "|" | sort | uniq | wc -l
```
</div>

2\. Which organism has the most copies of the 16S rRNA gene per genome?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
*Photobacterium damselae* has 21 copies

```
grep ">" data/raw/rrnDB-5.6_16S_rRNA.fasta | cut -f 1,2 -d "|" | sort | uniq -c | sort -r | cut -f 1 -d "|" | head
```
</div>


3\. After the species *coli*, which *Escherichia* species has the most genomes represented in the rrnDB?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
*Escherichia* with 6868

* 957 coli
* 15 albertii
* 5 fergusonii
* 1 sp.
* 1 marmotae

```
grep ">Escherichia" data/v19/rrnDB.align | cut -f 1,2 -d "|" | sort | uniq | cut -f 1 -d "|" | cut -f 2 -d "_" | sort | uniq -c | sort -r
```
</div>
