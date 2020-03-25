---
layout: page
title: Code Club
---

**Code Club** is an opportunity for people to work together to learn to program and analyze data better. In each session a facilitator will propose a problem or concept to learn, provide background material, disperse participants to work on the material together, and then bring people back together to provide feedback and debrief. The entire session should not last more than one hour. For people that cannot participate live via Zoom, edited content will be posted to YouTube for others to participate asynchronously.

This activity requires that participants take on a growth mindset and be mindful that not everyone has the same proficiency or experiences that they have. We assume that anyone participating in a Code Club is willing to adhere to our [code of conduct](code-of-conduct).

Although this may evolve over time, the primary language used in these Code Clubs will be [R](http://www.academichermit.com/2020/03/23/Why-R.html) and we will use RStudio as the primary interface for our activities. To participate, you will need internet access, R, Rstudio, and the tidyverse package. Here are some [instructions](setup-instructions) to help you get set up and ready for your first Code Club.

<ul class="post-preview">
	{% assign sorted = site.code_club | sort: 'date' | reverse %}

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
