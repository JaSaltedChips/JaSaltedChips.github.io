---
layout: default
title: "C++"
---
## C++

A collection of C++ programming notes, covering key concepts, and best practices: for quick reference and daily learning.

<ul>
  {% for post in site.posts %}
    {% if post.categories contains 'cpp-techniques' %}
      <li><a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%B %d, %Y" }}</li>
    {% endif %}
  {% endfor %}
</ul>