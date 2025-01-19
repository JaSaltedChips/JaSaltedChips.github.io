---
layout: default
title: "Personal Growth"
---
# Personal Growth Blogs

Here’s everything I’ve written about self-improvement:

<ul>
  {% for post in site.posts %}
    {% if post.url contains '/personal-growth/' %}
      <li><a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%B %d, %Y" }}</li>
    {% endif %}
  {% endfor %}
</ul>
