---
layout: post
title: "Customizing bash to improve reproducibility"
blurb: "Creating aliases and packing more information into our prompt"
author: "PD Schloss"
date: 2020-08-13 11:55
comments: false
start_hash: eda3a3fb
youtube: ZUDBDnZEjuo
---

In past episodes of Code Club we haven't spent much time talking about how to customize your bash environment. Perhaps you've already found how you can change the background and font colors using the themes that are built into the software. If you're going to spend a lot of time looking at the screen, it's nice to be able to create a pleasant working environment. Besides these aesthetic customizations, we can also customize the behavior of our command line environment to make it easier to interact with the command line.

In today's episode we'll see how we can modify the hidden `.bashrc` file that lives in your home directory. First, we'll customize our command line environment by creating aliases that are words that stand in for other things. For example, if there are arguments that you always use when running a command, you can put that into an alias. By default R will prompt you to save your session and offer to load data from a previous session. Both practices are problematic from a reproducibility stand point because you may lose track of what variables were created previously. But we can create an alias that always tells R to turn off those features. Second, we'll see how we can customize our prompt. I'll share with you the code used to generate a prompt that will tell us the status of our project's repository. In the exercises, I'll encourage you to make your own customizations.

An important point to remember in customizing our environment is that we don't want those customizations to impact our actual analysis. This is because if you get my code and my code depends on the customizations, but I don't give you those customization, then your version of the analysis will break. For example, imagine if were I create an alias for `sed` that is called `sed`, but runs `sed -E`. I might code my bash scripts in a way that assumes I always have access to the enhanced regular expressions. Now, you get a hold of my code and try to run those scripts. But it doesn't work for you. Why? Because you don't have the same `sed` alias that I have. We always want to be mindful of how much our bash scripts depend on our aliases.

Today's video won't do anything with the data from the project that we've been working on over the recent episodes. So, even if you're only watching this video to learn more about customizing your command line environment and don't know what a 16S rRNA gene is, you should get a lot out of today's video. Please take the time to follow along on your own computer and attempt the exercises. Don't worry if you aren't sure how to solve the exercises, at the end of the video I will provide solutions. If you haven't been following along but would like to, please check out the notes below where you'll find instructions on catching up, reference notes, and links to supplemental material. You can find [my version of the project](https://github.com/SchlossLab/Schloss_rrnAnalysis_mSphere_2020) on GitHub.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Pat's fancy command prompts

This code originally from [Karl Broman](https://kbroman.org) who was kind enough to share it with Pat

```bash
# color prompt to include branch information
function color_my_prompt {
  local __user_and_host="\[\033[01;32m\]\u@\h"
  local __cur_location="\[\033[01;34m\]\W"
  local __git_branch='`git branch 2> /dev/null | grep -e ^* | sed -E  s/^\\\\\*\ \(.+\)$/\(\\\\\1\)\ /`'
  local __prompt_tail="\[\033[35m\]$"
  local __last_color="\[\033[00m\]"

  RED="\[\033[0;31m\]"
  YELLOW="\[\033[0;33m\]"
  GREEN="\[\033[0;32m\]"

  # Capture the output of the "git status" command.
  git_status="$(git status 2> /dev/null)"

  # Set color based on clean/staged/dirty.
  if [[ ${git_status} =~ "working directory clean" ]]; then
      state="${GREEN}"
  elif [[ ${git_status} =~ "working tree clean" ]]; then
      state="${GREEN}"
  elif [[ ${git_status} =~ "Changes to be committed" ]]; then
      state="${YELLOW}"
  else
      state="${RED}"
  fi

  export PS1="$__user_and_host $__cur_location ${state}$__git_branch$__prompt_tail$__last_color "
}

# Tell bash to execute this function just before displaying its prompt.
PROMPT_COMMAND=color_my_prompt
```

## Important things to remember

* A good practice is to make a backup copy of your `.bashrc` and `.bash_profile` files before you start modifying them.
* You can load the modified files either by restarting the command line interface or by running

```bash
source ~/.bash_profile
```

or

```
source ~/.bashrc
```

### Alias

Syntax:

`alias alias_name="commands"`

Example:

`alias R="R --no-save --no-restore"`


### Customizing the prompt

* `PS1` is the environmental variable that defines what the prompt looks like
* `PROMPT_COMMAND` is the environment variable that tells bash which command to run before generating the prompt
* Useful meta characters for defining elements of the prompt
	* `\h`: Host
	* `\u`: Username
	* `\W`: Current working directory
	* `\w`: Full path to current directory
	* `\d`: Date
	* `\n`: Newline
	* `\t`: Time
* Don't forget to add a space following the actual prompt character (e.g. `$`)
* As an example, here's what the default prompt looks like

	```bash
	PS1="\h:\W \u$ "
	```
* [Here](https://code.tutsplus.com/tutorials/how-to-customize-the-command-prompt--net-20586) and [here](https://www.cyberciti.biz/tips/howto-linux-unix-bash-shell-setup-prompt.html) are several other tutorials on customizing your prompt


### Specific notes for Mac OS X about .bash files
* When `Terminal.app` is started, it runs `.bash_profile` first. You will want to call `.bashrc` from within your `.bash_profile` using something like...

	```
	[[ -s ~/.bashrc ]] && source ~/.bashrc
	```

* `Terminal.app` has a preferences tab that will allow you to customize the appearance of your environment
* With Catalina (OS X 10.15), the default shell script has become `zsh`. For the most part, the differences between bash and zsh should be minimal. You can tell `Terminal.app` to use `bash` in your `Terminal.app` preferences by going to the "General" tab and selecting the "Command" radio button and putting `/bin/bash` in the field for "Shell opens with".

### Specific notes for Linux and Windows 10 about .bash files
* When the command line interface opens, it will run `.bashrc` first, the `.bash_profile` file is not used


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

1\. Create an alias for `rm` that prompts you whenever you are about to delete a file. Normally this would be achieved using `rm -i <file_name>`. Where could this cause problems with our bash scripts?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```
alias rm="rm -i"
```

There are places in our bash scripts where we do some "garbage collection" with `rm`. We likely don't want to be asked about deleting stuff if we're automating it. If we want to keep this alias, then we need to go back and add `-f` to our `rm` commands.
</div>


2\. Create an alias called `lsl` that runs `ls -lth` whenever it is used

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```bash
alias lsl='ls -lth'
```
</div>


3\. The customized prompt outputs my user and computer name (i.e. user and host). To free up some room in the prompt, I'd like to remove that information. How would you modify the `color_my_prompt` function to do this?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
Modify the `export PS1` line to be

```bash
export PS1="$__cur_location ${state}$__git_branch$__prompt_tail$__last_color "
```
</div>
