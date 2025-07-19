---
layout: default
title: "Finance"
---
# Finance Blogs

Here’s everything I’ve written about self-improvement:

<ul>
  {% for post in site.posts %}
    {% if post.url contains '/finance/' %}
      <li><a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%B %d, %Y" }}</li>
    {% endif %}
  {% endfor %}
</ul>
