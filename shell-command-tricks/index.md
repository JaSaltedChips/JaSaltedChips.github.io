---
layout: default
title: "Shell Commands"
---
# Shell Commands

There are so many shell commands I keep forgetting or just can’t seem to remember. This is my go-to spot to store all of them, so I can easily come back when I need them — and you can too, if you ever need one!

<ul>
  {% for post in site.posts %}
    {% if post.url contains '/shell-command-tricks/' %}
      <li><a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%B %d, %Y" }}</li>
    {% endif %}
  {% endfor %}
</ul>
