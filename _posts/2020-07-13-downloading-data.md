---
layout: post
title: "Downloading data from the command line"
author: "PD Schloss"
date: 2020-07-13 11:55
blurb: "Who needs a browser, let's get our data directly"
comments: false
---

In the last episode of Code Club we downloaded the files that we will be using to determine to what degree inter- and intra-genomic variation limit the ability to interpret amplicon sequence variants (ASVs) as a biologically coherent entity that proponents of ASVs claim. We did this in the context of learning about GitHub flow (aka Git Flow). One problem with how we did it was that we needed to use a browser to navigate to a website to download the files. This is fine if we only need to do it once, but if we had to do it for multiple files, it could be a pain. Imagine that there is a directory of 100 files you want to download. Downloading those files manually would be a major pain. Our approach from last week would also be pretty challenging if we were working on a high performance computer that doesn't have a browser window. For today's code club we'll use GitHub flow to see how we can download the files we got last week using tools from the command line. In a future episode we'll gain a greater appreciation for this approach because it becomes easier to automate the process if the files are updated or if ours get corrupted.

Please take the time to watch today's episode, follow along on your own computer, and attempt the exercises. Don't worry if you aren't sure how to solve the exercises, at the end I will provide solutions.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/vldQ_oV6b70" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

The reference notes and links that follow are a supplement to the material in the video. The exercises and their solutions are provided at the bottom.


## Installations

### Mac OS X

We will use [Homebrew](https://brew.sh) to install `wget`. Mac OS X is kind of like Linux, except it doesn't quite have everything Linux does and even if it does have what we want, they tend to be old or weird versions. As the Homebrew documentation says, "Homebrew installs the stuff you need that Apple (or your Linux system) didn’t." `wget` is one of those missing packages that we need to install. First, we'll install Homebrew

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

You'll need to provide your password. You may be asked a few questions along the way and you might be asked to press "ENTER". Do what it asks. This might take a little time the first go through; be patient.

Then we'll install `wget`

```bash
brew install wget
```

### Windows

If you're using the [Ubuntu Linux BASH shell for Windows 10](https://itsfoss.com/install-bash-on-windows/), you're in good shape.


## Today's commands

### `wget`

`wget` allows us to automate the direct downloading of files to wherever we tell it. An alternative is `curl`. For whatever reason, I prefer `wget`.

In it's simplest form, the following will download `silva.seed_v138.tgz`

```bash
wget https://mothur.s3.us-east-2.amazonaws.com/wiki/silva.seed_v138.tgz
```

There are many different options that you can use with `wget` to do different things (see `wget --help`). We won't do it here, but know that you can download entire directories and can provide a username and password for protected files.

For our purposes, there are a few options we want to know about. The generic example above has a few related problems that are not ideal. First, if the file already exists on our local computer, it will re-download the file and it will also append `.1` to the end of the file name. Second, it doesn't let us designate where we want to put the file. We'll add a few options

* `--no-clobber` / `-nc`:  if the file is already present, move on
* `--directory-prefix=PREFIX` / `-P`:	download to a specified directory


We can add these to the example above

```bash
wget --no-clobber --directory-prefix=data/references https://mothur.s3.us-east-2.amazonaws.com/wiki/silva.seed_v138.tgz
```

If you run that command twice, you'll see that we get a message that the file already exists in our desired directory and that it won't retrieve the file again. What we have above is a verbose and descriptive way of writing the command. We can replace the options that start `--` with the abbreviated name that starts with `-`

```bash
wget -nc -P data/references https://mothur.s3.us-east-2.amazonaws.com/wiki/silva.seed_v138.tgz
```

Again, we're only scratching the surface on what we can do with `wget`. Generally googling "wget <what do you want to do>" will help you find examples of doing whatever you want to do with `wget`.


### `unzip`/`zip`

`unzip` allows us to un-zip files that have been compressed using `zip`. These files typically end in `.zip`. Zipped files are more common on Windows computers than on Mac OS X or Linux. With `zip`/`unzip` you can compress multiple files together. You can decompress a zipped file like this

```BASH
unzip data/raw/rrnDB-5.6.tsv.zip
```

Unfortunately, that will output the file to our current location, rather than to `data/raw/`. We can output the decompressed file to `data/raw/` using the `-d` argument. To be safe, you probably also want to include the `-n` argument to make sure that you are not overwriting existing files. This can be a problem if your zipped file contains a file named `README.md` (ours does not) and there is a different `README.md` file in your `data/raw/` directory

```BASH
unzip -n -d data/raw/ data/raw/rrnDB-5.6.tsv.zip
```


### `gunzip`/`gzip`

`gunzip` is an alternative to `unzip` that works with files that have been compressed using `gzip`. These files typically end in `.gz`. They are more common in Mac OS X and Linux than in Windows. One important difference between `.gz` and `.zip` files is that a `.gz` file is compressed version of a single file where as `.zip` files can be a compressed collection (i.e. "archive") of files. Another tools that are used to compress/decompress files that are frequently used on Mac OS X and Linux is `bzip2`/`bunzip2`. Unlike `unzip`, `gunzip` will output the decompressed file in the directory that the compressed file was located. It will also remove the compressed file unless you tell it otherwise (i.e. use `-k`). We'll be using `gunzip` with `tar`.


### `tar`

`tar` allows us to bundle files together into a "tarball" and it also allows us to unbundle them. Tarballs are not compressed. These are commonly found in Mac OS X and Linux and typically end in `.tar`. Because they are not compressed, `tar` has built in ability to also compress/decompress files using `gzip`/`gunzip` (and `bzip2`/`bunzip2`). Tarballs that are compressed with `gzip` typically end in `.tar.gz` or `.tgz`. Beware of [tar bombs](https://en.wikipedia.org/wiki/Tar_(computing)#Tarbomb), which when expanded overwrite your other local files and directories. The best defensive practice is to output the extracted files to a new directory and then move them where you want them, if they need to be moved. It's pretty rare to see a non-compressed tarball, so I'll show how to decompress and untar a compressed tarball

```BASH
mkdir data/references/silva.seed_v138/
tar xvzf data/references/silva.seed_v138.tgz -C data/references/silva_seed/
```

Let me unpack these two lines of code. First, we'll create `data/references/silva_seed/` to store the output of `silva.seed_v138.tgz`. Second, the `tar` command has four arguments:

* `x` - extract files (alternatively, `c` would be use to create a tarball)
* `v` - verbose: write out what is going on
* `z` - `gunzip` or `gzip` the file if you are opening or creating the tarball
* `f` - what follows is the file name

Finally, the `-C` flag tells `tar` where to send the output. You should notice that the files are written to `data/references/silva.seed_v138/`. If we had not used the `-C` flag then the files would end up in our project root directory. The `README.md` file from the tarball would have written over the `README.md` file in our project root directory. The same thing would have happened if we had used `-C` to output to `data/references/`. This is why it's a good idea to untar your files to an empty directory. There are a variety of other approaches you can use to avoid writing over your pre-existing files, but this is [generally considered the most reliable approach](http://www.linfo.org/tarbomb.html).


## Exercises

1\. What arguments would you give `wget` to supply a username and password? What argument would you use if you would rather `wget` ask you your password?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
```
--user=USER                 set both ftp and http user to USER
--password=PASS             set both ftp and http password to PASS
--ask-password              prompt for passwords
```
</div>

2\. How could you list the contents of `rrnDB-5.6.tsv.zip` without decompressing it? How about the contents of `silva.seed_v138.tgz`?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
Zip file...

```BASH
unzip -l data/raw/rrnDB-5.6.tsv.zip
```

Tarball...

```BASH
tar tvf data/references/silva.seed_v138.tgz
```
</div>


3\. Finish and push Issue 6 to your version of the repository

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
</div>
