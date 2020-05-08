---
layout: page
title: Code Club
---

**Code Club** is an opportunity for people to learn to program and analyze data better. In each session we'll provide background material, disperse participants to work on several exercises, and then come back to share solutions and debrief. The entire session should not last more than one hour. Videos of Pat Schloss presenting the material are posted to YouTube for people to engage with asynchronously.

Although this may evolve over time, the primary language used in these Code Clubs will be [R](http://www.academichermit.com/2020/03/23/Why-R.html) and we will use RStudio as the primary interface for our activities. To participate, you will need internet access, R, Rstudio, and the tidyverse package. Here are some [instructions](setup-instructions) to help you get set up and ready for your first Code Club.

<ul class="post-preview">
	{% assign sorted = site.posts | sort: 'date' | reverse %}

  {% for post in sorted %}
    <li>
      <span class="post-date">{{ post.date | date: "%b %-d, %Y" }}</span>
      <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
			{% if post.blurb != nil %}
			<p>{{ post.blurb }}</p>
			{% else %}
		      {{ post.excerpt }}
			{% endif %}
    </li>
  {% endfor %}
</ul>
