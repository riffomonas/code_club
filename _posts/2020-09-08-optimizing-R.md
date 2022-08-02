---
layout: post
title: "Why is our R script so sloooooowwwwwwwww?"
blurb: "We'll speed up the R script that we wrote in the last episode. Should we?"
author: "PD Schloss"
date: 2020-09-08 11:55
comments: false
start_hash: 56cf29e9e4f53
youtube: ttJCwzfsAc8
---

In the last episode of Code Club we learned how to write an executable R script. That script converted a very wide data frame with ~15000 columns into a long data frame with only 3 columns. R struggles with wide data frames and we saw that when we ran our script. It took a minute to read in the wide data frame, but only took a second to read in the same data in from a tidy format. Most people would be satisfied that they have the data in a better format and move on with the rest of their project. Not me! I'd like to use this example to show you how you can learn what steps in your code are bottlenecks. I'll also get you to think about the trade off between your time or "programmer time" and the time it takes your code to run or "execution time".

One of my pet peeves are blog posts of people ripping on R or ripping on Python. They'll pull out some test like how long it takes to calculate a mean. They'll then run that test using implementations in different languages. Because they don't know how to write code in the language they're trying to bash, they often present an unfair comparison. Also lost in the conversation is that the differences are relatively minor compared to the cost required to learn the favored language. If you can spot these straw man arguments then you likely already understand the strengths and weaknesses of the language you use. That's great, because then you can avoid the weaknesses when you write your own code!

R is designed to be efficient for you to write. For example, if I want to read in a file, I can use the `read_tsv` function. But, in another language, like C++, I would have to write a lot more code. The tradeoff is often the time it takes to execute the command. The R version would likely be considerably slower to run than the C++ version. Thinking back on that example of how long it takes to calculate the mean of a bunch of numbers, you could make your own function in R to calculate the mean. But, it will likely be slow. For most examples, the differences we're discussing are on the order of micro or nanoseconds for calculating a mean. Alternatively, you could use R's built in `mean` function, which is actually written in C or C++. This will likely be as fast as actually doing it in C or C++. It's way beyond the scope of what we're doing here, but you can use a package called RCpp to write C++ code in R to optimize steps that are too slow for your needs.

This all brings up an important point that I mentioned earlier. A lot of your R code may only be run a few times. For example, the code we wrote in the last episode read in the wide data frame and outputted it as a tidy data frame. It took maybe 2 minutes to run. Say I have to run it 5 times over the course of my debugging and execution of the project. That's 10 minutes total. Maybe I run 10 times. That's still less than an hour. How long is it going to take you to figure out how to speed it up? Maybe an hour or two? Is it worth going from 2 minutes to 11 seconds to run your code if it takes you an hour or two to refactor the code?

Sometimes. Today, I'm going to spend a bit of time showing you an alternative to functions like `read_tsv` and `pivot_longer` that are far faster, but have a less intuitive syntax. These functions will come from the `data.table` package. I would say it's worth learning these alternatives if you are going to be working with really large files or files that are wide. I say this, because the next time you run into this scenario, maybe you'll remember to use these alternative functions and will be able to pull out these tools to make your code more efficient. But, if the improvement is some slick programming trick that you're unlikely to need again, then maybe I'd say meh. It's slick, but has it gained you any time? Beyond seeing how to use some new functions from `data.frame`, you'll also learn how to profile your code so that you can find the bottlenecks in your code that you might want to refactor to make the code more efficient.

Even if you're only watching this video to learn more about R and don't know what a 16S rRNA gene is, I'm sure you'll get a lot out of today's video. Please take the time to follow along on your own computer. If you haven't been following along but would like to, please check out the notes below where you'll find instructions on catching up, reference notes, and links to supplemental material. You can find [my version of the project](https://github.com/SchlossLab/Schloss_rrnAnalysis_mSphere_2020) on GitHub.



<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



## Important things to remember

* Don't spend hours fixing something that only costs you seconds
* [Options for finding bottlenecks](https://adv-r.hadley.nz/perf-measure.html)
  - `system.time`: will report the elapsed (i.e. wall), user (i.e. time used by CPU to run command), system (i.e. time used by the operating system)
  - `profvis`: runs a chunk of code and periodically checks R to see what it's doing. Once the chunk has completed, it produces visual output telling you how long R spent doing each task
  - `bench`: will run a chunk of code a specified number of times and output a summary of those iterations. Has a lot of nice features baked into it like visualizations
* [Options for speeding things up](https://adv-r.hadley.nz/perf-improve.html)
  * Find a faster package (e.g. `fread` vs `read_tsv`)
  * Minimize nesting of loops
  * Minimize assignments within loops; create data structure outside of loop
  * Avoid making copies of data, especially within loops
  * Vectorize code
  * Use Rcpp to rewrite R code in C++




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
