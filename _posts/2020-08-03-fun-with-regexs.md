---
layout: post
title: "Fun with regular expressions in sed and grep"
blurb: "Expanding our bash toolset"
author: "PD Schloss"
date: 2020-08-03 13:55
comments: false
start_hash: 56ae068
youtube: yORbSp7Fe10
---

Since starting this project 10 episodes ago, we have yet to really leave the command line interface. You've learned a lot of bash syntax - `mkdir`, `cd`, `ls`, `pwd`, `touch`, `rm`, `rmdir`, `nano`, `git`, `make`, `sed`, `|`, `if`, `mv`, and probably a few others. If you have all those down, then you're in great shape and are likely seeing the value of using these bash commands to automate a reproducible workflow. But, if these still seem a bit challenging to you, don't fret! We're going to spend a few more episodes in bash to strengthen our familiarity with these commands and my general workflow.

In today's episode, we'll see many of those commands and some new ones to help solve a problem I found in our analysis. In the last two episodes, we used special patterns called "regular expressions" with `sed` to extract information from our file names and paths. If you did the exercises in the `sed` episode, I showed how you can run `sed` on the contents of a file rather than its name. But, `sed` isn't the only place that we can use regular expressions in bash. There's another, probably more popular tool called `grep` where we can use regular expressions. Heck, the name [grep is short for](https://en.wikipedia.org/wiki/Grep) "globally search for a regular expression and print matching lines".

After the last episode, I was looking back through our files and noticed that mothur had changed our sequence names because the names had spaces in them. Have I mentioned how horrible spaces are for bioinformatics work?! I also noticed that although most of our sequences start and end at the coordinates that we trimmed them to, there are a few for each region that don't. In those cases, mothur starts the sequence with a series of periods to indicate missing data. Later on, we might decide to toss those sequences because they're weird. I'd prefer to have those be hyphens to represent gap characters. Instead of opening these files in a text editor and replacing all the spaces with underscores or replacing the periods with hyphens, we can fix the information using `sed`. Along the way we'll learn a few extra commands to keep things interesting. These are the commands that I often use to diagnose problems or do simple analyses of data in my files.

Even if you're only watching this video to learn more about bash commands and don't know what a 16S rRNA gene is, I'm sure you'll get a lot out of today's video. Please take the time to follow along on your own computer and attempt the exercises. Don't worry if you aren't sure how to solve the exercises, at the end of the video I will provide solutions. If you haven't been following along but would like to, please check out the notes below where you'll find instructions on catching up, reference notes, and links to supplemental material. You can find [my version of the project](https://github.com/SchlossLab/Schloss_rrnAnalysis_mSphere_2020) on GitHub.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Important things to remember

### Getting help
For many commands, using the command name followed by `--help` or `-h` will bring you to a help page (e.g. `wget --help`). For others, you might need to use `man` or `info` followed by the command name (e.g. `man grep` or `info grep`). Many commands will get you help by either approach with the `man`/`info` output being more complete. In the `man`/`info` interface you can use the arrow keys to work through the document line by line or the space bar scroll a page at a time.


### Resources on regular expressions in bash
* Bash Guide for Beginners ([Chapter 4](https://www.tldp.org/LDP/Bash-Beginners-Guide/html/chap_04.html))
* [RegexOne](https://regexone.com)
* `^` and `$` - anchors that match at the beginning or end of the line
* `*` - matches the preceding character zero or more times
* `.` - matches any character
* `\.` - matches the `.`
* `\-` - matches the `-`
* `[AC]` - matches either an "A" or "C" in the line

### Common `grep` arguments
* `-c`: count the number of lines
* `-v`: return lines that don't match
* `-E`: extended regular expressions


### Common `wc` arguments
* default output: number of lines, words, and characters for the specified file
* `-c`: count the number of characters
* `-w`: count the number of words
* `-l`: count the number of lines


### Common `head`/`tail` arguments
* `-n`: number of lines to output. The default is 10



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

1\. How many of the sequences in `data/v19/rrnDB.align` have ambiguous bases (i.e. Ns) in them?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
grep -v ">" data/v19/rrnDB.align | grep -c "N"
grep -v ">" data/v19/rrnDB.align | grep "N" | wc -l
</div>

2\. How many of the full length sequences in `data/v19/rrnDB.align` contain the standard forward primer to amplify the V3 region (CCTACGGGAGGCAGCAG) or the V4 region (GTGCCAGCMGCCGCGGTAA)? Feel free to use `.` to represent the degenerate bases. Remember that the `*` represents the previous character occurring zero or more times. As a bonus, see if you can figure out how to modify your regular expression to represent [degenerate bases](https://www.bioinformatics.org/sms/iupac.html).

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
grep "C-*C-*T-*A-*C-*G-*G-*G-*A-*G-*G-*C-*A-*G-*C-*A-*G" data/v19/rrnDB.align | wc -l
grep "G-*T-*G-*C-*C-*A-*G-*C-*.-*G-*C-*C-*G-*C-*G-*G-*T-*A-*A" data/v19/rrnDB.align | wc -l
grep "G-*T-*G-*C-*C-*A-*G-*C-*[AC]-*G-*C-*C-*G-*C-*G-*G-*T-*A-*A" data/v19/rrnDB.align | wc -l
</div>

3\. The fasta sequence headers contain five fields separated by pipe characters (i.e. `|`). Can you generate a file that contains the five fields separated by commas (i.e. `,`)? Be sure to remove the `>`. To stretch yourself, figure out how to give the five fields names without using a text editor.

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
echo "organism_name,genome_accession,sequence_accession,chromosome,genome_coordiantes" > header_table.csv
grep ">" data/v19/rrnDB.align | sed "s/|/,/g" | sed "s/>//" >> header_table.csv
</div>
