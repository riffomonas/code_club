---
layout: post
title: "Using the BFG Repo-Cleaner to delete files from your GitHub history (CC295)"
blurb: "Removing big and sensitive data from git and GitHub"
author: "PD Schloss"
date: 2024-07-22 9:00
comments: false
youtube: _XUxVMQn5xY
start_hash: 21daf1747139f5dadcbf6104e1fdd1310e0454e1
end_hash: 7a58ad168fc034cbc66576c6f22c5f643baf53cc
---

Every now and then we accidentally push a large file or a file with sensitive data like a password to GitHub using git. In this Code Club, Pat shows how you can use the BFG Repo-Cleaner to delete a large file that he pushed to his repository in a previous episode. He talks about the difference between using "git rm" and the BFG Repo-Cleaner. He'll also show how to install the java runtime environment along with the BFG Repo-Cleaner. This episode is part of an ongoing effort to develop an R package that implements the naive Bayesian classifier.

<iframe style="margin: 0 auto;display:block;" width="560" height="315" src="https://www.youtube.com/embed/{{ page.youtube }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Code

You can browse the state of the repository at the

* [beginning of the episode](https://github.com/riffomonas/phylotypr/tree/{{page.start_hash}})
* [end of the episode](https://github.com/riffomonas/phylotyprr/tree/{{page.end_hash}})
