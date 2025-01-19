---
layout: default
title: "Insightful Inklings"
---
## Insightful Inklings

Is a collection of handy commands and everyday discoveries — a go-to spot for one seeking quick tips and practical knowledge. It’s a treasure chest of unsorted yet invaluable insights. Here's everything that I have collected:

<ul>
  {% for post in site.posts %}
    {% if post.categories contains 'shell-command-tricks' %}
      <li><a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%B %d, %Y" }}</li>
    {% endif %}
  {% endfor %}
</ul>
