---
layout: post
title: "Building logic into bash scripts with if-else statements"
blurb: "A smarter bash script"
author: "PD Schloss"
date: 2020-07-30 11:55
comments: false
start_hash: 9697028
youtube: xrSQsw0EclM
---

In the last episode we used `sed` to write regular expressions that allowed us to parse file names and paths to make our bash scripts a little easier to use. We also tweaked our Makefile to take advantage of those changes by using some of make's automatic variables. `sed` is a powerful tool for writing bash scripts and working with file names. But that episode was really a bit of a set up for today's episode. Today, I want to build some logic into our bash scripts.

Right now, we have full length 16S rRNA gene sequences. They were extracted from genome sequences. Sure there are sequencing platforms that will generate full length 16S rRNA gene sequences. However, the Illumina MiSeq platform is far more common and people are using it to sequence much shorter fragments of the gene that are a few hundred nucleotides long. If I want to look at how amplicon sequence variants perform using current sequencing approaches, then I need to extract the more commonly used regions of the gene. We'll also look at full length sequences, but I need to build some logic into a new script that will extract the coordinates of my alignment that correspond to the region I give the script. This will allow us to simulate creating amplicon sequence variants (ASVs) for different subregions of the 16S rRNA gene. In today's episode of Code Club, we're going to use if-else statements in bash to insert this type of logic into our bash scripts. We'll also see how we can use if-else statements to make sure that everything has been going swimmingly.

Even if you aren't analyzing 16S rRNA gene sequences or even interested in microbiology, I'm sure you'll get a lot out of today's video. Please take the time to follow along on your own computer and attempt the exercises. Don't worry if you aren't sure how to solve the exercises, at the end of the video I will provide solutions. If you haven't been following along but would like to, please check out the notes below where you'll find instructions on catching up, reference notes, and links to supplemental material. You can find [my version of the project](https://github.com/SchlossLab/Schloss_rrnAnalysis_mSphere_2020) on GitHub.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Important things to remember

### Syntax

* basic `if` statment

	```bash
	if [[ <query> ]]
	then

	<do stuff>

	fi
	```

* `if...else if` statement

	```bash
	if [[ <query> ]]
	then

	<do stuff>

	elif [[ <query> ]]
	then

	<do stuff>

	fi
	```

* `if...else if...else` statement

	```bash
	if [[ <query> ]]
	then

	<do stuff>

	elif [[ <query> ]]
	then

	<do stuff>

	else

	<do stuff>

	fi
	```

### Query syntax

These are the more commonly used operators for `if` statements [ht: [linuxize](https://linuxize.com/post/bash-if-else-statement/#test-operators)]

```
STRING1 = STRING2 - True if STRING1 and STRING2 are equal.
STRING1 != STRING2 - True if STRING1 and STRING2 are not equal.
INTEGER1 -eq INTEGER2 - True if INTEGER1 and INTEGER2 are equal.
INTEGER1 -gt INTEGER2 - True if INTEGER1 is greater than INTEGER2.
INTEGER1 -lt INTEGER2 - True if INTEGER1 is less than INTEGER2.
INTEGER1 -ge INTEGER2 - True if INTEGER1 is equal or greater than INTEGER2.
INTEGER1 -le INTEGER2 - True if INTEGER1 is equal or less than INTEGER2.

-n VAR - True if the length of VAR is greater than zero.
-z VAR - True if the VAR is empty.
-h FILE - True if the FILE exists and is a symbolic link.
-r FILE - True if the FILE exists and is readable.
-w FILE - True if the FILE exists and is writable.
-x FILE - True if the FILE exists and is executable.
-d FILE - True if the FILE exists and is a directory.
-e FILE - True if the FILE exists and is a file, regardless of type (node, directory, socket, etc.).
-f FILE - True if the FILE exists and is a regular file (not a directory or device).
```



### Helper commands and variables

* `$?` the value returned from the last command you ran
* `exit #` leave the script at that point and return the value of `#`; 0 means success



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

1\. We included the rules and logic to output the alignment file for the full length sequences (i.e. v19) and sequences for the V4 region (i.e. v4). Write the rules and logic for the V3-V4 (i.e. v34) and V4-V5 (i.e. v45) regions. The start coordinate for the V3 region is 6428 and the end coordinate for the V5 region is 27659. Update the code and close issue 12.


2\. We unzipped the files from the rrnDB and then touched the extracted file to update its timestamp. Create and close an issue to modify the code in `code/get_rrndb_files.sh` so that the timestamp is only touched if the extraction was successful.


3\. We have yet to create a rule to run the `code/install_mothur.sh` script. Create and close an issue to update the timestamp if it successfully installs and include it as a dependency in the rules that use mothur.
